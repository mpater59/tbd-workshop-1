IMPORTANT ❗ ❗ ❗ Please remember to destroy all the resources after each work session. You can recreate infrastructure by creating new PR and merging it to master.

![img.png](doc/figures/destroy.png)

0. The goal of this phase is to create infrastructure, perform benchmarking/scalability tests of sample three-tier lakehouse solution and analyze the results using:
* [TPC-DI benchmark](https://www.tpc.org/tpcdi/)
* [dbt - data transformation tool](https://www.getdbt.com/)
* [GCP Composer - managed Apache Airflow](https://cloud.google.com/composer?hl=pl)
* [GCP Dataproc - managed Apache Spark](https://spark.apache.org/)
* [GCP Vertex AI Workbench - managed JupyterLab](https://cloud.google.com/vertex-ai-notebooks?hl=pl)

Worth to read:
* https://docs.getdbt.com/docs/introduction
* https://airflow.apache.org/docs/apache-airflow/stable/index.html
* https://spark.apache.org/docs/latest/api/python/index.html
* https://medium.com/snowflake/loading-the-tpc-di-benchmark-dataset-into-snowflake-96011e2c26cf
* https://www.databricks.com/blog/2023/04/14/how-we-performed-etl-one-billion-records-under-1-delta-live-tables.html

2. Authors:

   ***enter your group nr: 4***

   ***link to forked repo: https://github.com/mpater59/tbd-workshop-1***

   - Mikołaj Pater
   - Aleksandra Harasimik-Buraczewska
   - Marcin Kurowski

3. Sync your repo with https://github.com/bdg-tbd/tbd-workshop-1.

4. Provision your infrastructure.

    a) setup Vertex AI Workbench `pyspark` kernel as described in point [8](https://github.com/bdg-tbd/tbd-workshop-1/tree/v1.0.32#project-setup) 

    b) upload [tpc-di-setup.ipynb](https://github.com/bdg-tbd/tbd-workshop-1/blob/v1.0.36/notebooks/tpc-di-setup.ipynb) to 
the running instance of your Vertex AI Workbench

5. In `tpc-di-setup.ipynb` modify cell under section ***Clone tbd-tpc-di repo***:

   a)first, fork https://github.com/mwiewior/tbd-tpc-di.git to your github organization.

   b)create new branch (e.g. 'notebook') in your fork of tbd-tpc-di and modify profiles.yaml by commenting following lines:
   ```  
        #"spark.driver.port": "30000"
        #"spark.blockManager.port": "30001"
        #"spark.driver.host": "10.11.0.5"  #FIXME: Result of the command (kubectl get nodes -o json |  jq -r '.items[0].status.addresses[0].address')
        #"spark.driver.bindAddress": "0.0.0.0"
   ```
   This lines are required to run dbt on airflow but have to be commented while running dbt in notebook.

   c)update git clone command to point to ***your fork***.

 


6. Access Vertex AI Workbench and run cell by cell notebook `tpc-di-setup.ipynb`.

    a) in the first cell of the notebook replace: `%env DATA_BUCKET=tbd-2023z-9910-data` with your data bucket.


   b) in the cell:
         ```%%bash
         mkdir -p git && cd git
         git clone https://github.com/mwiewior/tbd-tpc-di.git
         cd tbd-tpc-di
         git pull
         ```
      replace repo with your fork. Next checkout to 'notebook' branch.
   
    c) after running first cells your fork of `tbd-tpc-di` repository will be cloned into Vertex AI  enviroment (see git folder).

    d) take a look on `git/tbd-tpc-di/profiles.yaml`. This file includes Spark parameters that can be changed if you need to increase the number of executors and
  ```
   server_side_parameters:
       "spark.driver.memory": "2g"
       "spark.executor.memory": "4g"
       "spark.executor.instances": "2"
       "spark.hadoop.hive.metastore.warehouse.dir": "hdfs:///user/hive/warehouse/"
  ```


7. Explore files created by generator and describe them, including format, content, total size.

   ***Files desccription***

Po wygenerowaniu zbioru danych w folderze /tmp/tpc-di pojawiły się następujące pliki:

![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/14808292-59fb-4d71-867b-c847105181dd)

Rozmiary podfolderów:

![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/31e00ac6-8a6a-4468-8b73-2ad58d6db4ae)

Najważniejszymi plikami jakie powstały po wygenerowaniu zbioru danych to były pliki .csv oraz .txt. Powstały przy tym 3 foldery: Batch1, Batch2 oraz Batch3. W folderze Batch1 znajdują głównie pliki tekstowe .txt (lub bez żadnego formatu) oraz odpowiadające im pliki z końcówką i rozszerzeniem _audit.csv. Te pierwsze pliki zajmują najwięcej miejsca i w nich są przechowywane dane z bazy danych, natomiast te pliki .csv zawierają informacje na temat kolumn dla każdego z rekordu z bazy danych. W folderach Batch2 i Batch3 jest to samo, jedynie przechowywane tam dane są o wiele mniejszego rozmiaru.

Dodatkowo po wykonaniu tego kroku został wygenerowany raport:

![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/509257c3-10ef-4741-891f-fd906851795a)


8. Analyze tpcdi.py. What happened in the loading stage?

   ***Your answer***

Skrypt tpcdi.py był odpowiedzialny za zapisanie danych lokalnie w /tmp/tpc-di/Batch1 w naszym projektowym kubełku tbd-2024l-303946-data (funkcja upload_files() oraz get_stage_path(), która zwracała ścieżkę do pliku w kubełku).

![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/671562b8-480b-43bc-bc23-35aa89ecb6a5)

Dodatkowo, przy pomocy plików .csv te wszystkie dane zostały zapisane w Spark DataFrame (funkcja load_csv() oraz save_df()). Ostatecznie zostały utworzone poniższe tabele:

![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/b7626e8b-9319-43dd-98cf-e7b43d2321e1)

9. Using SparkSQL answer: how many table were created in each layer?

   ***SparkSQL command and output***

Wykorzystany kod:

```python
databases = []
result = {}
for value in spark.sql("show databases").collect():
    databases.append(value.namespace)

for database in databases:
    spark.sql(f"use {database}")
    tables = spark.sql("show tables")
    result[database] = tables.count()

for key, value in result.items():
    print(f"Layer {key} - Number of tables: {value}")
```

Rezultat:

![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/45492d12-d2c7-4dd9-bf89-eb2cbd64cd41)



10. Add some 3 more [dbt tests](https://docs.getdbt.com/docs/build/tests) and explain what you are testing. ***Add new tests to your repository.***

   ***Code and description of your tests***

1. Test (dim_customer__unique_customer.sql) - Sprawdzanie czy wszyscy klienci w tabeli 'dim_customer' są unikalni.

```sql
select
    sk_customer_id,
    count(*) cnt
from {{ ref('dim_customer') }}
group by sk_customer_id
having cnt > 1
```

2. Test (fact_cash_transactions__unique_cash_transaction.sql) - Sprawdzanie czy wszystkie transakcje w tabeli 'fact_cash_transactions' są unikalne.

```sql
select
    sk_customer_id,
    sk_account_id,
    sk_transaction_date,
    transaction_timestamp,
    description,
    count(*) cnt
from {{ ref('fact_cash_transactions') }}
group by sk_customer_id, sk_account_id, sk_transaction_date, transaction_timestamp, description
having cnt > 1
```

3. Test (fact_cash_transactions__null_amount.sql) - Sprawdzanie czy wszystkie transakcje w tabeli 'fact_cash_transactions' mają przypisaną jakąś wartość dla kolumny 'amount'.

```sql
select
    sk_customer_id,
    sk_account_id,
    sk_transaction_date,
    count(*) cnt
from {{ ref('fact_cash_transactions') }}
where amount is null
group by sk_customer_id, sk_account_id, sk_transaction_date
having cnt > 0
```

Po dodaniu ich do naszego repozytorium, sprawdzono czy te testy wykonują się poprawnie:

![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/aee9acb3-87f3-4026-9d5b-6ff89e250f31)


11. In main.tf update
   ```
   dbt_git_repo            = "https://github.com/mwiewior/tbd-tpc-di.git"
   dbt_git_repo_branch     = "main"
   ```
   so dbt_git_repo points to your fork of tbd-tpc-di. 

12. Redeploy infrastructure and check if the DAG finished with no errors:

***The screenshot of Apache Aiflow UI***

![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/b54871eb-998c-4077-87fd-a61950bd2820)

