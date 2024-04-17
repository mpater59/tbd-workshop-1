IMPORTANT ❗ ❗ ❗ Please remember to destroy all the resources after each work session. You can recreate infrastructure by creating new PR and merging it to master.
  
![img.png](doc/figures/destroy.png)

1. Authors:

   ***enter your group nr: 4***

   ***link to forked repo: https://github.com/mpater59/tbd-workshop-1***

   - Mikołaj Pater
   - Aleksandra Harasimik-Buraczewska
   - Marcin Kurowski
   
2. Follow all steps in README.md.

3. Select your project and set budget alerts on 5%, 25%, 50%, 80% of 50$ (in cloud console -> billing -> budget & alerts -> create buget; unclick discounts and promotions&others while creating budget).

  ![img.png](doc/figures/discounts.png)

5. From avaialble Github Actions select and run destroy on main branch.
   
6. Create new git branch and:
    1. Modify tasks-phase1.md file.
    
    2. Create PR from this branch to **YOUR** master and merge it to make new release. 
    
    ***place the screenshot from GA after succesfull application of release***

   ![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/0d857771-a820-49a0-acf0-63391c33bf7a)


7. Analyze terraform code. Play with terraform plan, terraform graph to investigate different modules.

    ***describe one selected module and put the output of terraform graph for this module here***

   Terraform graph for Vertex AI Workbench module:
   ![terraform_graph_vertex_ai_workbench](https://github.com/mpater59/tbd-workshop-1/assets/32270817/8536ae46-7e11-426a-b34a-4748bc7bef91)

   Vertex AI Workbench is Jupyter notebook-based development environment for the data science workflow. In this terraform module we can see that two buckets are created, one to store configuration data, while the other contains a shell script ("notebook_post_startup_script.sh") that is used to launch the 
   appropriate images. Additionally, Terraform pulls a container image (default as "gcr.io/deeplearning-platform-release/base-cpu.py310:latest"), which is then run on an "e2-standard-2" machine. For our project, this machine is named as "tbd-2024l-303946-notebook" and we can use it to access Jupyter notebook.

   With these machines running, you can: run BigQuery with Jupyter notebook; run PySpark applications; proccess data by running notebooks on Dataproc clusters.
   
8. Reach YARN UI
   
   ***place the command you used for setting up the tunnel, the port and the screenshot of YARN UI here***

   used command: ***gcloud compute --project "tbd-2024l-303946" ssh --zone "europe-west1-d" "tbd-cluster-m" -- -L 8088:localhost:8088***

   In this case we needed to connect to the master node of the Dataproc cluster (named "tbd-cluster-m" in this project) and configure a tunnel for port 8088 to connect to the web UI.

   ***Screenshot of YARN UI***
   ![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/5a3b9da8-4914-4902-9c97-303ffff159f8)

   
9. Draw an architecture diagram (e.g. in draw.io) that includes:
    1. VPC topology with service assignment to subnets

       ![task9_1](https://github.com/mpater59/tbd-workshop-1/assets/32270817/0fc5ae98-3caa-436d-942d-1cb683c20c3a)

       There are 2 subnets in our project. The first one (subnet-01) is used to ensure connectivity between nodes for Dataproc and Vertex AI Notebook. The second subnet (composer-subnet-01) is used by Cloud Composer.

    2. Description of the components of service accounts

        - 611727548048-compute@developer.gserviceaccount.com (iac) - This SA manages connection between GitHub and Google Cloud service, it is responsible for distributing the access tokens.
        - tbd-2024l-303946-data@tbd-2024l-303946.iam.gserviceaccount.com (tbd-composer-sa) - This SA manages Cloud Composer environment, Dataproc clusters and jobs.
        - tbd-2024l-303946-lab@tbd-2024l-303946.iam.gserviceaccount.com (tbd-terraform)	- This SA is used for terraform activities, it allows for communication and management of the project infrastracture in Google Cloud form the terraform level. 

    3. List of buckets for disposal
    
        - tbd-2024l-303946-state - Bucket with saved terraform files
        - tbd-2024l-303946-code - Bucket with saved code files for Apache Spark
        - tbd-2024l-303946-conf - Bucket with saved "notebook_post_startup_script.sh" script
        - tbd-2024l-303946-data - Bucket with saved data from data-pipelines
    
    4. Description of network communication (ports, why it is necessary to specify the host for the driver) of Apache Spark running from Vertex AI Workbech
  
        By default there are 4 VM defined that run on the same subnet ("subnet-01" - 10.10.10.0/24):
        - JupyterLab Notebook VM - tbd-2024l-303946-notebook - 10.10.10.5
        - Master - tbd-cluster-m - 10.10.10.3
        - Worker 0 - tbd-cluster-w-0 - 10.10.10.4
        - Worker 1 - tbd-cluster-w-1 - 10.10.10.2
      
        In Apache Spark it is necessary to specify the host for the driver beacuse it is responsible for executing main() functions and creating the SparkContext. This driver needs to communicate with the worker nodes to divide tasks between them and the master node in order to monitor the status of these nodes and their working status. 

        In our project, driver is listing on port 30000 and block manager is listining on port 30001.

       Topology of Apache Spark Cluster:
       
       ![task9_2](https://github.com/mpater59/tbd-workshop-1/assets/32270817/666e0291-13f0-4bb7-bad8-fffa972d7c9e)


10. Create a new PR and add costs by entering the expected consumption into Infracost
For all the resources of type: `google_artifact_registry`, `google_storage_bucket`, `google_service_networking_connection`
create a sample usage profiles and add it to the Infracost task in CI/CD pipeline. Usage file [example](https://github.com/infracost/infracost/blob/master/infracost-usage-example.yml) 

   ***place the expected consumption you entered here***

  Added infracost-usage: [infracost-usage.yml](infracost-usage.yml)

   ```yaml
  version: 0.1
  resource_usage:
    google_artifact_registry_repository:
      storage_gb: 50 # Total data stored in the repository in GB
      monthly_egress_data_transfer_gb: # Monthly data delivered from the artifact registry repository in GB. You can specify any number of Google Cloud regions below, replacing - for _ e.g.:
        europe_west3: 20 # GB of data delivered from the artifact registry to europe-west3.
    google_storage_bucket:
      storage_gb: 500 # Total size of bucket in GB.
      monthly_class_a_operations: 1000000 # Monthly number of class A operations (object adds, bucket/object list).
      monthly_class_b_operations: 12500000 # Monthly number of class B operations (object gets, retrieve bucket/object metadata).
      monthly_data_retrieval_gb: 250 # Monthly amount of data retrieved in GB.
      monthly_egress_data_transfer_gb: # Monthly data transfer from Cloud Storage to the following, in GB:
        same_continent: 200 # Same continent.
        worldwide: 0 # Worldwide excluding Asia, Australia.
        asia: 0 # Asia excluding China, but including Hong Kong.
        china: 0 # China excluding Hong Kong.
        australia: 0 # Australia.
    google_service_networking_connection:
      monthly_egress_data_transfer_gb: # Monthly VM-VM data transfer from VPN gateway to the following, in GB:
        same_region: 100 # VMs in the same Google Cloud region.
        us_or_canada: 0 # From a Google Cloud region in the US or Canada to another Google Cloud region in the US or Canada.
        europe: 100 # Between Google Cloud regions within Europe.
        asia: 0 # Between Google Cloud regions within Asia.
        south_america: 0 # Between Google Cloud regions within South America.
        oceania: 0 # Indonesia and Oceania to/from any Google Cloud region.
        worldwide: 100 # to a Google Cloud region on another continent.
  ```

  Modified pull request workflow: [.github/workflows/pull-request.yml](.github/workflows/pull-request.yml)

  ```diff
      - name: Generate Infracost cost estimate baseline
        run: |
          infracost breakdown --path="." \
                              --format=json \
                              --out-file=/tmp/infracost-base.json \
+                             --usage-file infracost-usage.yml
      - name: Checkout PR branch
        uses: actions/checkout@v3
      - name: Generate Infracost diff
        run: |
          infracost diff --path="." \
                          --format=json \
                          --compare-to=/tmp/infracost-base.json \
                          --out-file=/tmp/infracost.json \
+                         --usage-file infracost-usage.yml
      - name: Post Infracost comment
  ```

   Disclaimer: The expected consumption that we entered was set arbitrarily, therefore the obtained costs of infrastracture consumption may be incorrect.

   ***place the screenshot from infracost output here***

   ![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/9a9ba20a-0f34-4a72-abba-803b718e257e)

   ![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/251bd5aa-e65e-49e5-a872-28ba1351e31c)


11. Create a BigQuery dataset and an external table using SQL
    
    ***place the code and output here***

    ![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/b0f5b3d4-1314-42f6-8ed0-f95b70ba0f9f)

    Code:
    ```sql
    CREATE SCHEMA IF NOT EXISTS demo OPTIONS(location = 'europe-west1');

    CREATE OR REPLACE EXTERNAL TABLE demo.shakespeare
      OPTIONS (
      
      format = 'ORC',
      uris = ['gs://tbd-2024l-303946-data/data/shakespeare/*.orc']);
    
    
    SELECT * FROM demo.shakespeare ORDER BY sum_word_count DESC LIMIT 5;
    ```
    Disclaimer: We used this code from README.md (with the bucket name changed) but before we could use it we first had to run fixed spark-job.py from task 13. Running this job allowed to save the appropriate data in .orc format to our bucket, which allowed us to execute the above BigQuery.
   
    ***why does ORC not require a table schema?***

    ORC files doesn't require a table schema because they contain metadata within the file itself. They can contain information about the table schema that BigQuery can read.

  
12. Start an interactive session from Vertex AI workbench:

    ***place the screenshot of notebook here***
    Used command: ***gcloud compute --project "tbd-2024l-303946" ssh --zone "europe-west1-d" "tbd-cluster-m" -- -L 8080:localhost:8080***
    
    ![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/33863228-120a-4763-abb5-ac07145b806e)

   
13. Find and correct the error in spark-job.py

    ***describe the cause and how to find the error***

    Modified file: [modules/data-pipeline/resources/spark-job.py](modules/data-pipeline/resources/spark-job.py)

    ```diff
    # change to your data bucket
    --- DATA_BUCKET = "gs://tbd-2024l-9910-data/data/shakespeare/"
    +++ DATA_BUCKET = "gs://tbd-2024l-303946-data/data/shakespeare/"
    
    spark = SparkSession.builder.appName('Shakespeare WordCount').getOrCreate()
    ```

    In this case, the error was that the spark-job.py file contained the wrong name of the bucket to which the data was to be saved after the job was completed.
    
    Used command to execute spark-job.py: ***gcloud dataproc jobs submit pyspark gs://tbd-2024l-303946-code/spark-job.py --cluster=tbd-cluster --region=europe-west1 --project "tbd-2024l-303946"***

    Partial output:
    ![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/53b1d7f5-f639-4f32-b00a-cd4cbd7db071)


14. Additional tasks using Terraform:

    1. Add support for arbitrary machine types and worker nodes for a Dataproc cluster and JupyterLab instance

    ***place the link to the modified file and inserted terraform code***

    Modified files:

    [main.tf](main.tf)

    [modules/dataproc/main.tf](modules/dataproc/main.tf)

    [modules/dataproc/variables.tf](modules/dataproc/variables.tf)

    [modules/vertex-ai-workbench/main.tf](modules/vertex-ai-workbench/main.tf)

    [modules/vertex-ai-workbench/variables.tf](modules/vertex-ai-workbench/variables.tf)

    Modified code: [Commit task 14.1](https://github.com/bdg-tbd/tbd-workshop-1/commit/59abfa53849eee3e9af6e682053d93c9243919b8)

    Disclaimer: We've also added the ability to manually set the number of worker node instances, all these changes can be made by editing the appropriate lines in the main.tf file. Another option would be to introduce additional variables in the variables.tf file and set the appropriate default values.
    
    2. Add support for preemptible/spot instances in a Dataproc cluster

    ***place the link to the modified file and inserted terraform code***

    Modified files:

    [main.tf](main.tf)

    [modules/dataproc/main.tf](modules/dataproc/main.tf)

    [modules/dataproc/variables.tf](modules/dataproc/variables.tf)

    Modified code: [Commit task 14.2](https://github.com/bdg-tbd/tbd-workshop-1/commit/5dc341f890d6892548a5eb021db34dfd4320b42b)
    
    3. Perform additional hardening of Jupyterlab environment, i.e. disable sudo access and enable secure boot
    
    ***place the link to the modified file and inserted terraform code***

    Modified files:
    
    [modules/vertex-ai-workbench/main.tf](modules/vertex-ai-workbench/main.tf)

    Modified code: [Commit task 14.3](https://github.com/bdg-tbd/tbd-workshop-1/commit/9534cb7e285b88e3fb06318ebc585e641e04dd2b)

    4. (Optional) Get access to Apache Spark WebUI

    ***place the link to the modified file and inserted terraform code***
    
    Modified files:
    
    [main.tf](main.tf)
    
    [modules/dataproc/main.tf](modules/dataproc/main.tf)

    Modified code:

    [Commit: Dataproc - enabled http port access](https://github.com/bdg-tbd/tbd-workshop-1/commit/903acc25c1b42b671c5a1e47f7fbef4ae38d3198)
    
    [Commit: Enabled google-beta](https://github.com/bdg-tbd/tbd-workshop-1/commit/7628b192de46c9cae441b3ef988a4110eead2c00)

    ![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/0bfd9973-7411-485a-8f33-5d8dafdcaad8)

    For this task, we wanted to use the HTTP address that is provided when running spark-shell on the master node. For this reason, we enabled http port access in the Dataproc terraform module (which additionally required the use of google-beta provider:  https://registry.terraform.io/providers/hashicorp/google/latest/docs/resources/dataproc_cluster#nested_endpoint_config). Unfortunately, we still couldn't access Apache Spark Web UI externally.

    ![image](https://github.com/mpater59/tbd-workshop-1/assets/32270817/44992687-c5dd-462c-911d-7c5d33110ed7)

    
    
