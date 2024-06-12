0. The goal of phase 2b is to perform benchmarking/scalability tests of sample three-tier lakehouse solution.

1. In main.tf, change machine_type at:

```
module "dataproc" {
  depends_on   = [module.vpc]
  source       = "github.com/bdg-tbd/tbd-workshop-1.git?ref=v1.0.36/modules/dataproc"
  project_name = var.project_name
  region       = var.region
  subnet       = module.vpc.subnets[local.notebook_subnet_id].id
  machine_type = "e2-standard-2"
}
```

and subsititute "e2-standard-2" with "e2-standard-4".

2. If needed request to increase cpu quotas (e.g. to 30 CPUs): 
https://console.cloud.google.com/apis/api/compute.googleapis.com/quotas?project=tbd-2023z-9918

3. Using tbd-tpc-di notebook perform dbt run with different number of executors, i.e., 1, 2, and 5, by changing:
```
 "spark.executor.instances": "2"
```

in profiles.yml.

4. In the notebook, collect console output from dbt run, then parse it and retrieve total execution time and execution times of processing each model. Save the results from each number of executors. 

5. Analyze the performance and scalability of execution times of each model. Visualize and discucss the final results.

Wykonano 3 testy dla 1, 2 oraz 5 instancji egzekutorów. W trakcie przetwarzania modeli przez Spark'a, zauważono w UI YARN'a, że zmieniała się liczba uruchomionych kontenerów oraz wykorzystywanych vCPU. Dla testu z uruchomioną 1 instancją egzekutora były wykorzystywane 2 kontenery oraz 2 vCPU. Dla testu z uruchomionymi 2 instancjami egzekutora były wykorzystywane 3 kontenery oraz 3 vCPU. Dla testu z uruchomionymi 5 instancjami egzekutora były wykorzystywane 6 kontenerów oraz 6 vCPU.

Następnie zapisano wszystkie wyniki do następujących plików tekstowych: [phase2b_test1_dbt_run_log.txt](dbt_run_logs/phase2b_test1_dbt_run_log.txt), [phase2b_test2_dbt_run_log.txt](dbt_run_logs/phase2b_test2_dbt_run_log.txt), [phase2b_test5_dbt_run_log.txt](dbt_run_logs/phase2b_test5_dbt_run_log.txt)

Na ich podstawie przygotowano arkusz Excel ([dbt_run_logs_results.xlsx](dbt_run_logs/dbt_run_logs_results.xlsx)), w którym zapisano wszystkie wyniki i stworzone odpowiednie wykresy, które znajdują się poniżej.

Zapis uzyskany wyników w tabeli:

![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/d2cfdd61-74ac-4dfc-b331-fd801cfca746)

Wykres przedstawiający łączny czas przetwarzania dla poszczególnego testu z różną liczbą uruchomionych instancji egzekutora:

![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/71d1faa7-1f4c-4cee-bb22-cbefa0f245dd)

Wykres przedstawiający czasy przetwarzania dla poszczególnego modelu tabeli w zależności od liczby uruchomionych instancji egzekutora:

![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/6485c105-7a46-4780-8cfa-efdb042a0fc6)

**Wnioski**

Porównując całkowite czasy przetwarzania można zaobserwować, że dla testu z uruchominą jedną instancją egzekutora czas przetwarzania był największy i dodanie kolejnej instancji egzekutora pozwoliło na zmniejszenie tego czasu przetwarzania o ponad połowę. Jednakże zysk pomiędzy uruchomionymi 2 oraz 5 instancjami egzekutora był bardzo niewielki. Dla pewności uruchomiono jeszcze raz test z 5 instancjami egzekutora (zebrane logi znajdują się w [phase2b_test5_v2_dbt_run_log.txt](dbt_run_logs/phase2b_test5_v2_dbt_run_log.txt)), jednakże uzyskane wyniki wciąż były do siebie bardzo zbliżone.

Na drugim wykresie przedstawiający czasy przetwarzania dla poszczególnych modeli można zaobserwować skąd się biorą te zbliżone wartości dla całkowitych czasów przetwarzania. W pierwszej kolejności można zauważyć, że dla modelu "demo_silver.daily_market" dodawanie kolejnych egzekutorów pozwalało na skalowalne zmniejszanie czasu jego przetwarzania. Jednakże takich modeli, gdzie czasy przetwarzania by się zmniejszały odpowiednio stopniowo względem liczby uruchomionych instancji egzekutora jest niewiele. Większość tych modeli była po prostu za mała, aby był jakiś zysk z większej liczby uruchomionych instancji egzekutorów, a po drugie, to nie każdy model się dobrze skalował z dużą liczbą uruchomionych instancji egzekutorów. Można to zaobserwować w szczególności na samych końcowych modelach (względem ich numeru indeksu, który się pojawiał w logach), gdzie najlepsze czasy przetwarzania osiągano dla testu z 2 uruchomionymi instancjami egzekutorów, a nie z 5 jakby mogło się to wydawać. Wynika to najprawdopodobniej ze specyfiki danego modelu, gdzie nie zawsze się da wiele rzeczy na raz zrównoleglić, a uruchamianie więcej egzekutorów niż to jest potrzebne może prowadzić do niechcianego zwiększenia czasu przetwarzania takich modeli.

Z tych wykresów można także zauważyć, że dla naszego przypadku najlepszym przypadkiem byłoby wykorzystywanie 2 uruchomionych instancji egzekutora, aby zapewnić jak najlepszy stosunek wydajność/koszty. Uruchomienie dodatkowego egzekutora zmniejszyło nam czas przetwarzania modeli o ponad połowę względem 1 egzekutora, a przez to, że w tym teście było tylko kilka takich większych modeli do przetworzenia, większość z nich była stosunkowo małych rozmiarów. To sprawia, że uruchamianie większej liczby egzekutorów niż 2 może nie być optymalne, bo zwiększamy tym sobie koszty zużycia vCPU, a nie daje to nam większego zysku. Gdyby było więcej modeli podobnych do "demo_silver.daily_market" do przetworzenia, to wtedy można byłoby wykorzystać większą liczbę egzekutorów, ponieważ powinno to się już wtedy dobrze skalować i odczuwalnie zmniejszyć czas ich przetwarzania. W przypadku mniejszych modeli to trzeba brać pod uwagę to, że uruchamianie większej liczby instancji egzekutorów niesie za sobą jakieś zwiększone koszty przy starcie przetwarzania, co w niektórych przypadkach może nam całkowicie pogorszyć wydajność przetwarzania, jak to można było zaobserwować w ostatnim teście z 5 egzekutorami.
