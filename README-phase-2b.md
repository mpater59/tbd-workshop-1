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

Wykonano 3 testy dla 1, 2 oraz 5 instancji egzekutorów. W trakcie przetwarzania modeli tabel przez Spark'a, zauważono w YARN'ie, że zmieniała się liczba uruchomionych kontenerów oraz wykorzystywanych vCPU. Dla testu z uruchomioną 1 instancją egzekutora były wykorzystywane 2 kontenery oraz 2 vCPU. Dla testu z uruchomionymi 2 instancjami egzekutora były wykorzystywane 3 kontenery oraz 3 vCPU. Dla testu z uruchomionymi 5 instancjami egzekutora były wykorzystywane 6 kontenerów oraz 6 vCPU.

Następnie zapisano wszystkie wyniki do następujących plików tekstowych: #TODO

Na ich podstawie przygotowano arkusz Excel (#ścieżka do pliku), w którym zapisano wszystkie wyniki i stworzone odpowiednie wykresy, które znajdują się poniżej:

Zapis uzyskany wyników w tabeli:

#TODO

Wykres przedstawiający łączny czas przetwarzania dla poszczególnego testu z różną liczbą uruchomionych instancji egzekutora:

#TODO

Wykres przedstawiający czasy przetwarzania dla poszczególnego modelu tabeli w zależności od liczby uruchomionych instancji egzekutora:

#TODO


   
