

*******************************************************************************************************************************************************************************************************************************************************

DISCLAIMER:

“IMPORTANT NOTICE: "Information contained and transmitted by this EMAIL including any attachment is proprietary to NPCI and is intended solely for the addressee/s, and may contain information that is privileged, confidential or exempt from disclosure under applicable law. Access to this email and/or to the attachment by anyone else is unauthorized. If this is a forwarded message, the content and the views expressed in this EMAIL may not reflect those of the NPCI. If you are not the intended recipient, an agent of the intended recipient or a person responsible for delivering the information to the named recipient, you are notified that any use, distribution, transmission, printing, copying or dissemination of this information in any way or in any manner is strictly prohibited.

If you are not the intended recipient of kindly delete this email from your system and inform the sender. There is no guarantee that the integrity of this communication has been maintained and nor is this communication free of viruses, interceptions or interference”.

********************************************************************************************************************************************************************************************************************************************************

Show quoted text
Show quoted text
Show quoted text
Seldon Core is an open-source platform that allows you to deploy, manage, and monitor machine learning models at scale. It's particularly well-suited for serving machine learning models in production environments. Seldon Core is built on top of Kubernetes, which provides robust container orchestration capabilities.

Here's an overview of how Seldon Core model serving works:

1. **Define your Model**: First, you need to define your machine learning model using the Seldon Core specifications. This involves creating a Docker container that encapsulates your model and implements the required endpoints for prediction.

2. **Create a Seldon Deployment**: Once your model container is ready, you define a Seldon Deployment YAML file that describes how to deploy your model. The YAML file includes information such as the model's container image, scaling options, and resource requirements.

3. **Deploy to Kubernetes**: With the Seldon Deployment YAML file in hand, you deploy it to a Kubernetes cluster using the standard Kubernetes deployment process. Seldon Core will create the necessary Kubernetes resources and start serving your model.

4. **Model Serving and Predictions**: Once the Seldon Deployment is running, Seldon Core exposes an API endpoint that allows you to make predictions using your model. You can send HTTP POST requests to this endpoint with input data, and the Seldon Core service will forward the data to your model container, receive the predictions, and return the results.

5. **Monitoring and Metrics**: Seldon Core provides built-in monitoring and metrics capabilities, which allow you to monitor the performance and health of your deployed models. You can track metrics such as request latency, throughput, and error rates.

6. **Scaling and Load Balancing**: Seldon Core also supports scaling your model serving based on demand. It can automatically scale the number of replicas running your model based on configurable thresholds and metrics.

Overall, Seldon Core simplifies the process of deploying machine learning models in production by leveraging Kubernetes' scalability and reliability features. It also provides additional tools for A/B testing, canary deployments, and other advanced deployment scenarios.

For detailed information on how to use Seldon Core and the specific configuration options, you should refer to the official Seldon Core documentation, as it may have evolved beyond my knowledge cutoff date (September 2021).
Show quoted text
## Introduction

**Rupay Fraud Model** makes predictions about a transaction that will be fraudulent or not, and stops the transaction if it is fraudulent. Rupay fraud model helps in regulating such types of fraud and preventing frauds from taking place through Rupay. Lets see how we built this Seldon Coremodel.


## Key steps
This are the following key steps followed :

1.  [Inboarding Kubeflow notebooks](#inboarding-kubeflow-notebooks)
2.  [DBT models for profiles](#dbt-models-for-profiles)
3.  [Dagster data pipeline](#dagster-data-pipeline)
4.  [Feast feature store](#feast-feature-store)
5.  [Kubeflow components and pipelines](#kubeflow-components-and-pipelines)
6.  [Seldon core model serving](#seldon-core-model-serving)

### Inboarding Kubeflow notebooks
---

> Kubeflow Notebooks provides a way to run web-based development environments inside your Kubernetes cluster by running them inside Pods in which development environment is available inside of Kubeflow (and which packages are installed) is determined by Docker image used to invoke the Notebook server and admins can provide standard notebook images for their organisation with all the required packages pre-installed.
>
> For more details about Kubeflow Notebooks, refer [link](https://git.npci.org.in/genius-dev/ml-engineering/drona/kubeflow-pipelines/-/wikis/Getting-started-with-Kubeflow-Notebooks).

Refer [this link](https://git.npci.org.in/genius-dev/ml-engineering/drona/kubeflow-pipelines/-/wikis/Inboarding-Kubeflow-Notebooks) to find the process of creating Kubeflow Notebooks.


### DBT models for profiles
---

> DBT enables analytics engineers to transform data in their warehouses by writing select the statements. DBT handles turning these select statements into tables and views. DBT does the transformation (T) in extract, load, transform (ELT) processes.
>
> For more information about DBT models, refer [link](https://git.npci.org.in/genius-dev/data-engineering/dbt-dagster/aeps/-/wikis/DBT-Technical-Document).

DBT run reads base table from Trino, generate profile with features and save features back in Trino. Profile provides the model with specific characteristics or dimensions i.e., features of card holder and merchant which is used to form pattern to detect fraudulent transactions.
After inboarding kubeflow Notebooks with specified configurations, DBT models for profile generation are created.
You can find process of creating DBT models below.
1.  Navigating to `/home/jovyan/namespace-volume` and install the below specified requirements.

| Requirement | Version | Command |
| ------ | ------ | ------ |
| dbt-core | 1.4.5 | pip install -i https://centos79:dtcDr7TQBiMdWFIdrY@repo.npci.org.in/repository/python-remote/simple --user dbt-core==1.4.5 |
| dbt-extractor | 0.4.1 | pip install -i https://centos79:dtcDr7TQBiMdWFIdrY@repo.npci.org.in/repository/python-remote/simple --user dbt-extractor==0.4.1 |
| dbt-trino | 1.4.1 | pip install -i https://centos79:dtcDr7TQBiMdWFIdrY@repo.npci.org.in/repository/python-remote/simple --user dbt-trino==1.4.1 |

2.   Run `dbt init [project name]` from the command line to set up your new project. During project initialization, dbt creates sample model files in your project directory to help you start developing quickly. Here we run `dbt init frm_dbt` to create *frm_dbt* project.

![Screenshot_from_2023-07-18_12-25-53](uploads/f47844ea9dda19d79748132b73172543/Screenshot_from_2023-07-18_12-25-53.png)

3.  Navigate to `/home/jovyan/namespace-volume/[project name]` or `/.dbt` folder in home for configuration of DBT model properties using *profiles.yml*. Find the *profiles.yml* used for running DBT models of Rupay fraud model below. Make sure the below configurations are properly added as below content.

> ```
> frm_dbt:
>   outputs:
>     dev:
>       type: trino
>       method: none
>       user: trino
>       password: [password]
>       database: hive
>       host: 10.8.178.51
>       port: 30003
>       schema: rupaydb
>       threads: 8
>     prod:
>       type: trino
>       method: none  # optional, one of {none | ldap | kerberos}
>       user: [prod_user]
>       password: [prod_password]  # required if method is ldap or kerberos
>       database: [database name]
>       host: [hostname]
>       port: [port number]
>       schema: [prod_schema]
>       threads: [1 or more]
>   target: dev
> ```

4.  Let's create a example model i.e., test_frm.sql for better understanding, Navigate to `/home/jovyan/namespace-volume/[project name]/models` and create a text file with .sql extension as show below.

![sqlmodel.drawio](uploads/d9d87a6bca73d7c0374349346875720a/sqlmodel.drawio.png)

5.  Data Tables created by DBT models. Run `dbt run -m [model name].sql` from the command line in the folder where we created *profiles.yml* to execute DBT model. Find the description about tables below.\
[Click here to know more about running DBT models.](https://git.npci.org.in/genius-dev/data-engineering/dbt-dagster/aeps/-/wikis/Process-flow-for-DBT-Model-execution)

| Table | Description |
| ------ | ------ |
| rupay_test.distri_nonfraud | This table contains random transactions from `rupaydb.rupay_ft_dbt` of count `62743` from January 1st 2023 to March 31st 2023,`159227` from year 2022, `178029` from year 2021 such that entries are present in the table from every month in selected years.  |
| rupaydb.rupay_fraud_ft | This table is of only fraudulent transactions from january 1st 2021 to march 31st 2023. |
| rupaydb.non_fraud_21jan23mar_txns_v1 | This table is of only non-fraudulent transactions from rupay_test.distri_nonfraud table.   |
| rupaydb.rupay_train_base_table_jan21mar23_v12 | This table is union of `rupaydb.rupay_fraud_ft` and `rupaydb.non_fraud_21jan23mar_txns_v1` with `1` and `0` flags for fraudulent and non-fraudulent transactions respectively. |
| rupay_test.train_history_table_pan_v2 | This table contains past 180 days transactions for distinct card number and date in `rupaydb.rupay_train_base_table_jan21mar23_v12` table. |
| rupaydb.frm_unique_cus_profile_v2 | This table contains profile features for distinct card numbers from `rupay_test.train_history_table_pan_v2` table. |
| rupaydb.frm_unique_mer_profile.sql | This table contains profile features for distinct card accept id (ie., merchant id) from `rupay_test.train_history_table_pan_v2` table. |

### Dagster data pipeline
---

> Dagster is an orchestrator that's designed for developing and maintaining data assets, such as tables, data sets, machine learning models, and reports.You declare functions that you want to run and the data assets that those functions produce or update. Dagster then helps you run your functions at the right time and keep your assets up-to-date.
>
> Dagster is built to be used at every stage of the data development lifecycle - local development, unit tests, integration tests, staging environments, all the way up to production.

1.   Navigate to `/home/jovyan/namespace-volume` and run `pip install -r requirement.txt --user` to install specified requirements needed for dagster pipeline project creation and  updates source code of Dagster pipeline in Dagster repository for scheduling DBT Models and Feast Jobs.

2.   After setting up with requirements to run Dagster Pipeline project. First, Initialize the dagster project by running `dagster project scaffold --name my-dagster-project` from the command line and Configure repository.py according to the DBT project and schedule the jobs. Refer [this link](https://git.npci.org.in/genius-dev/data-engineering/dbt-dagster/rupay/tree/Rupay/RuPay_DBT/Rupay_Dagster_Pipeline) to find the dagster pipeline project of rupay fraud model.
*  Let us understand what happens in this dagster pipeline project. In the folder `/home/jovyan/namespace-volume/Rupay_Dagster_Pipeline/Rupay_Dagster_Pipeline/assets`, we create Python files of different tasks like DBT model execution, feast materlize... and by creating Python files i.e., jobs which call this tasks in `../Rupay_Dagster_Pipeline/jobs` folder and repository.py is configured to schedule all this jobs.

3.  Dagster Jenkins pipeline is triggered and DBT Runs and Feast materialize are scheduled in Dagster.

4.  User can monitor the Dagster runs through Dagster UI.

### Feast feature store
---

> Feast (Feature Store) is an open source feature store for machine learning. A feature store is a central repository for storing, managing, and serving features, which are the individual data elements used as inputs to machine learning models. Feast is the fastest path to manage existing infrastructure to productionize analytic data for model training and online inference. Feast can be seen as an architecture for managing features in ML pipelines. Benefits of Feast include: Feature versioning, Data integration, Low latency serving, Scalability, Metadata management, Feature engineering, Model training and serving intergration.

1.  Required features from the profile table are fetched and update the features in Feast repository.

* Clone the Feature store repository into your branch. Refer [this link](https://git.npci.org.in/genius-dev/ml-engineering/drona/kubeflow-pipelines/-/wikis/Cloning-repository-into-a-branch-and-other-git-operations) for cloning and other git related operations.

* Run `pip install -r requirement.txt --user` to install specified requirements needed to execute Feast through Kubeflow Notebook from `/home/jovyan/namespace-volume` folder. You can find the requirement.txt file [here](https://git.npci.org.in/genius-dev/ml-engineering/drona/feature-store/blob/master/requirement.txt).


*  Next, Work on feature store with Feast by creating python file(or files) for setting up feature store which contains various configuration of the Datasource and Entity definitions, Feature view design for different data (profile data), defining the feature service and the features it will serve and other necessary settings and *YAML* which is used to define the configuration and settings for the feature store and its associated components and stage your work by adding files in git.


Find the *YAML* file contents used in this model for configuartion below for reference.  
> ```yaml
> project: feature_store
> registry: registry.db
> provider: local
> offline_store:
>     type: feast.infra.offline_stores.contrib.trino_offline_store.trino.TrinoOfflineStore
>     host: 10.8.178.54
>     port: 30003
>     catalog: hive
>     connector:
>         type: hive
>         file_format: parquet
> online_store:
> #    path: online_store.db
>    type: redis
>    key_ttl_seconds: 60
> #    connection_string: "10.8.178.51:6379"
>   # redis_type: redis_cluster
>    connection_string: "10.43.216.224:6379,password=admin"
> ```


*  Git add, commit, push and create a pull request to the upstream repository to add the updated or created files. Refer [this link](https://git.npci.org.in/genius-dev/ml-engineering/drona/kubeflow-pipelines/-/wikis/Cloning-repository-into-a-branch-and-other-git-operations) to learn about those git operations.

2.  Dagster Pipeline executes feast apply (For feature view and feature service). This execution is done through Dagster UI.

> NOTE: Find the python code used to create Dagster pipeline which executes feast apply in [this link](https://git.npci.org.in/genius-dev/data-engineering/dbt-dagster/aeps/tree/trijal/aeps_dagster/ops_model) in developing AePS fraud model for reference.


3.  After Dagster pipeline triggered, the Feast materializes the features in Redis. The Dagster pipeline is triggered either manually or when it is scheduled.

### Kubeflow components and pipelines
---
> Component is the basic unit of execution logic in KFP. A component is a named template for how to run a container using an image, a command, and arguments. Components may also have inputs and outputs, making a component a computational template, analogous to a function. The output of a component will be the input of the next component and so on to form a pipeline. A pipeline component is a self-contained set of user code, packaged as a Docker Image , that performs one step in the pipeline. For example, a component can be responsible for data preprocessing, data transformation, model training, train test split and so on. A pipline function may have many inputs and outputs.\
> A Kubeflow pipeline is a description of an ML workflow, including all the components in the workflow and how they combine in the form of a graph. A pipeline function instantiates components as tasks and uses them to form a graph.

* Refer [this link](https://git.npci.org.in/genius-dev/ml-engineering/drona/kubeflow-pipelines/-/wikis/About-Kubeflow) to know more about Kubeflow components, tasks and pipelines.

1.  Update the source code for Component store and Kubeflow pipeline in Component store and Kubeflow Pipelines repository respectively.

*  This code is typically written in Python and specifies the sequence of data processing and modelling steps to be executed thus, defining the machine learning pipeline. This code is updated into Component store repository and this components are executed as tasks in Kubeflow pipeline. We cloned the existing [Component store](https://git.npci.org.in/genius-dev/ml-engineering/drona/component-store.git) and [Kubeflow pipelines](https://git.npci.org.in/genius-dev/ml-engineering/drona/kubeflow-pipelines.git) repositories and used it for building Component store and Kubeflow pipelines for building Rupay fraud model.

*  Run `pip install -r requirement.txt --user` to install specified requirements needed to execute Feast through Kubeflow Notebook from `/home/jovyan/namespace-volume` folder. You can find the Component store requirement.txt file [here](https://git.npci.org.in/genius-dev/ml-engineering/drona/component-store/blob/master/requirements.txt) and Kubeflow pipeline requirement.txt file [here](https://git.npci.org.in/genius-dev/ml-engineering/drona/kubeflow-pipelines/blob/master/requirements.txt).

2.  Navigate to `/home/jovyan/namespace-volume/kubeflow-pipelines/pipelines/rupay` folder and configure `rupay_config.py` which helps to select numerical columns from profile table.

3.  We manually run `rupay_training_pipeline.py` in the folder=/home/jovyan/namespace-volume/kubeflow-pipelines/pipelines/rupay ([link](https://git.npci.org.in/genius-dev/ml-engineering/drona/kubeflow-pipelines/tree/hiteshm/pipelines/rupay)) using the Kubeflow notebook with VS code custom image which creates the pipeline version in Kubeflow and triggers a run with given parameters. Refer [this link](https://git.npci.org.in/genius-dev/ml-engineering/drona/kubeflow-pipelines/-/wikis/Inboarding-Kubeflow-Notebooks) to learn about inboarding this notebook.
![runkfpipe.drawio](uploads/1c52629b38c8d9ed671d6a4cf95e07be/runkfpipe.drawio.png)


    *  Kubeflow pulls the component base images from Nexus repository.

        *  A "component base image" is a pre-built container image that contains the necessary dependencies and libraries for a specific step in the pipeline. Kubeflow fetches these component base images from a Nexus repository, which is a repository for storing and managing container images.
\
&nbsp;

    *  Kubeflow Downloads Feast Feature info from Feast bucket.

        *  We know that "Feast" is a data ingestion and feature store system. In this step, Kubeflow fetches feature information (data related to specific features or variables used for machine learning) from a feast bucket.
\
&nbsp;


    *  Kubeflow prepares the dataset using feast by joining the features with base table in Trino.

        *  Here, Kubeflow uses Feast to process the downloaded feature information and combines it with the base table i.e., rupay financial transactions table using Trino. This step prepares the dataset with all the required features for model training.
\
&nbsp;
    *  Kubeflow uses Kubeflow bucket for passing datasets and artifacts between components.

        *  The Kubeflow bucket is a storage location used to pass data and artifacts and metrics between different steps or components of the pipeline. It acts as a temporary holding area for data in the pipeline. It acts as a temporary holding area for data in the pipeline.
\
&nbsp;
    *  Kubeflow Writes Artifacts, Model and Metadata into ML flow server for permanent storage.

        *  As MLflow is used for managing models and experiments in a scalable and collaborative manner in the ML lifecycle and refer to [this Mlflow documentation link](https://mlflow.org/docs/latest/index.html) to learn more about MLflow . In this step, Kubeflow stores the artifacts (output files), the trained model, and metadata (information about the model's performance, training settings, etc.) into the MLflow server for permanent storage and versioning.
\
&nbsp;
    *  Mlflow stores the Artifacts in Mlflow Bucket.

        *  MLflow saves the artifacts produced by the pipeline into a dedicated MLflow bucket, a storage location where all pipeline artifacts are stored systematically.
\
&nbsp;
    *  User can monitor the pipeline run through Kubeflow UI.

        *  As a user, you can view the progress and status of the pipeline run using the Kubeflow User Interface (UI). This allows you to see how each step is performing and if there are any errors or issues.
\
&nbsp;
    *  User can track the Artifacts, compare metrices and metadata and register model into model store through Mlflow UI.

        *  Through MLflow UI, you can explore the artifacts produced by the pipeline, compare metrics (such as model performance metrics), and view metadata associated with each model run. Thia helps you analyze and assess the quality of the model and its outputs.
\
&nbsp;

Find the detailed flowchart of above processes below.

![KFcomp_n_pipeline.drawio](uploads/f7dd1e513b67d5238782768666512f4a/KFcomp_n_pipeline.drawio.png)

Find the picture of Kubeflow pipeline of rupay fraud model below.

![pipelinekf1.drawio](uploads/8f2e86eb66eb93731adbd0099dfa6708/pipelinekf1.drawio.png)

### Seldon core model serving
---

> Seldon Core is an open-source platform that allows you to deploy, manage, and monitor machine learning models at scale. It's particularly well-suited for serving machine learning models in production environments. Seldon Core is built on top of Kubernetes, which provides robust container orchestration capabilities.
> Overall, Seldon Core simplifies the process of deploying machine learning models in production by leveraging Kubernetes' scalability and reliability features. It also provides additional tools for A/B testing, canary deployments, and other advanced deployment scenarios. Refer the Seldon Core documentation to learn more.

1. User updates the Seldon Core Service code in model service repository.

2. Model Service Jenkins pipeline deploys the service code to seldon server.

3. Seldon Fetches the models from Mlflow Model Store and starts Server for serving the model as API.

4. On Request, Seldon service fetch Features from Redis online feature store and give back model response.

From: KETHAVATH PRANEETH NAYAK <cs20btech11025@iith.ac.in>
Sent: 27 July 2023 17:11
To: Kethavath Nayak <praneethnayak.k@npci.org.in>
Subject: Re: Flowchart creation
 
WARNING: This mail has come from outside. Please verify sender, attachment and URLs before clicking on them.

Show quoted text
Disclaimer:-
This footer text is to convey that this email is sent by one of the users of IITH. So, do not mark it as SPAM.

Show quoted text
##  Introduction

**NPCI** helps the banks carry out secure payment transactions by identifying and detecting fraud that can take place on the Rupay Ecom and POS systems. At a high level, there are primarily two ways that Rupay fraud occurs.

* **Social Engineering frauds**
* **Technical Frauds**

NPCI makes predictions about a transaction that will be fraudulent or not, and stops the transaction if it is fraudulent. This helps in regulating such types of fraud and preventing frauds from taking place through Rupay.

---
### Social Engineering Frauds
---

#### 1. Deceiving card holder to provide card details and authentication details like OTP :
Fraudsters tricks a card holder by Impersonating,Phishing, Vishing, Pretexting which involves a false scenario to trick individuals into revealing their confidential details.


---
### Technical Frauds
---
#### 1. Card Not Present (CNP) Fraud :
This type of fraud occurs when card details are used for online or over-the-phone transactions without physical presence of the card where fraudsters obtain card details through phishing, data breaches and many other ways.
Generally, They may use this information to make fraudulent purchases on e-commerce websites or other online transactions.
#### 2. Card Skimming :
Fraudsters use skimming devices to steal card information when the card is swiped at unautherized device, such as a compromised point-of-sale terminal. The stolen data is then used to create counterfeit Rupay cards or for online transactions.

## Rupay Fraud Model

The analytics team has developed the Rupay fraud model, which assists banks to prevent fraudulent transactions.
So, After a customer falls victim to fraud involving their Rupay card, they promptly report the incident to their bank. Subsequently, the bank categorizes the fraudulent transactions within the Enterprise fraud risk management (eFRM) portal. The NPCI analytics team thoroughly examines the data and collaborates with the business team to figure out the underlying patterns or trends.

![Flow1_frm.svg](uploads/756fce73015373abaf84b1dcb8c8f7c6/Flow1_frm.svg)


## Step-by-step process of Rupay Fraud Model flowchart

1.  When we receive an input transaction from eFRM, the JSON format is validated and, depending on the validation, a Response Fraud Score is assigned to the transaction.

2.  After successful validation in JSON format, we fetch the user profiles of user and merchant from Redis.

3.  Then the transaction goes to AI/ML model for prediction.

4.  After being predicted by the fraud model, the transaction is forwarded to the Response Formatter, which produces two response formats as below.

      a. Internal format - format used for internal purposes in the team.

      b. External format – format used to send for eFRM.

5.  If the transaction is not predicted to be fraudulent, we assign a fraud score of 300 and send a response to the eFRM team.

6.  If a transaction is predicted to be fraudulent, the following conditions will occur as below.

     a.      If the fraud flag is set to false, then we assign a fraud score is equal to 800.

     b.      The fraud flag is set to true, then the following conditions will occur as below.


         i. Check global thresholds and if it’s not breached, we assign fraud score between
            851-899 and the transaction gets declined.

        ii. If the global threshold is reached, we assign a fraud score of 801-850 and the
            transaction gets alerted to acquirer banks.

# Modelling approach of Rupay fraud model

The following are the methods and procedures used to build Rupay fraud model which is used to help card holders to detect and prevent fraudulent transactions.

![frm_model_build.drawio__1_.svg](uploads/d551b4e1da4b8b1b7b61f325e3c754dd/frm_model_build.drawio__1_.svg)

From: Kethavath Nayak <praneethnayak.k@npci.org.in>
Sent: 27 July 2023 19:40
To: KETHAVATH PRANEETH NAYAK <cs20btech11025@iith.ac.in>
Subject: Re: Flowchart creation
 
Show quoted text

