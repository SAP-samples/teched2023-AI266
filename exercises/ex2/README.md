#  Advanced Hands on lab

# Use case

In this Hands on lab we will be training a [house price dataset](https://scikit-learn.org/stable/datasets/real_world.html#california-housing-dataset) available in scikit-learn. Then use the generated house price model to create a deployment and do online inferencing

# Introduction:

## AICORE:

SAP AI Core is a service designed to handle the execution and operations of your AI assets in a standardized, scalable, and hyperscaler-agnostic way.

SAP AI Core provides the infrastructure to productize your AI applications and services. You can train and deploy AI models cost-efficiently at scale and integrate them into your business applications with the help of a standardized AI interface. 

AI scenarios can be developed using any open source ML framework. SAP AI Core provides full lifecycle management support such as content deployment using the GitOps principle. It is built around state-of-the-art open source solutions such as Argo Workflows and KFServing

With SAP AI Core, customers gain peace of mind as they benefit from a complete service that packages all dependencies and hides the complexity of building your own training and serving productive environment

## AI Launchpad

SAP AI Launchpad is a multitenant software as a service (SaaS) application in SAP Business Technology Platform to access and manage your AI use cases. Connect to multiple runtimes, including SAP AI Core, and streamline the AI lifecycle management of your use cases. Leverage SAP AI Launchpad to monitor your models and continuously improve their performance.


## Terminology

### Scenario and Executable

1. A Scenario corresponds to a AI Use case that you want to train (and/or) deploy
2. To make your scenario available in aicore you have to create either workflow template/serving template
3. Workflow template and Serving template correspond to a Training Executable and Serving Executable respectively
4. Executables/templates are hosted under a scenario
5. An executable includes information about the training input and output, the container images to be used, and the required infrastructure resources. SAP AI Core provides several preconfigured infrastructure bundles called resource plans that differ from one another depending on the number of CPUs, GPUs and the amount of memory.
6. Executables are design time entities, meaning you design them once and then they can be instantiated multiple times

### Tenant and Resource Group Relationship

<left><img src="./images/Tenant_RG_Relationship.png" alt="Tenant Resource Group relationship" style="width: 800px;"/></left>

The tenant concept provides a separation of data and workspaces of customers in an SAP system.

1. A Tenant can have many resource groups. Resource groups are isolated housing units for executions and deployments. With this isolation one resource group cannot access the executions and deployments of another resource group.

2. Datasets, models, configuration, executions and deployments are all resource group scoped.

3. Datasets and models belong to an AICORE entity called artifacts.

4. Configuration binds executable with artifacts and parameters to create a unique object with which we can create an instance of execution or deployment.

5. Execution uses the configuration with training executable to train a model.

6. Deployment uses the configuration with serving executable to serve the model.

## AICORE End to End Workflow

1. Subscribe to AICORE service in BTP
2. Onboard a git repository with workflow/serving templates
3. Create a docker registry secret
4. Create a Resource Group
5. Register your Object Store storage
6. Register a Training Dataset
7. Create a Configuration for your Training Executable
8. Train on your dataset
9. Create a Configuration for your Serving Executable
10. Serve the model as a web service

We have already subscribed to the AICORE service in BTP, you can find the details in the launchpad instance shared below.


<center><img src="./images/AICORE_workflow.png" alt="AICORE E2E workflow" style="width: 1000px;"/></center>

&nbsp;

# AIL Overview

AI Launchpad URL - [https://dcom-blr-handson.ai-launchpad.prodintern.eu-central-1.aws.apps.ml.hana.ondemand.com](https://dcom-blr-handson.ai-launchpad.prodintern.eu-central-1.aws.apps.ml.hana.ondemand.com)


There are 3 sub-applications to work with in the AI Launchpad app. 

1. Workspaces app
2. SAP AICORE Administration app
3. ML Operations app

### Workspaces App

Used to add AI API compliant runtimes to AI Launchpad app to manage the lifecycle of AI assets. The aicore service key (available after a subscription to AICORE service in BTP) is preconfigured in the AI Launchpad URL shared above


### SAP AICORE Administration app

Used to perform administration activities for your SAP AI Core runtime

### ML Operations app

Used to trigger executions and deployments, watch the logs and visualize metrics after conducting AI experiments.

&nbsp;

# Hands on steps

### 1. Dockerhub account

1. You need a Docker repository to store your AI code in the form of Docker images. When you create an execution or a deployment later AICORE will run the code from the docker image.
2. Sign up for [dockerhub](https://hub.docker.com/signup), skip this step if you already have a docker hub account
3. After logging in to dockerhub, click on account settings -> security
4. Create a new access token (this will be used in the docker registry creation step). select all default values and save the token locally.

### 2. AI Code

The code is available at $HOME/dcom-blr-handson folder locally (it is cloned from https://github.tools.sap/MLF/dcom-blr-handson)

1. For the house price prediction use case the code is available in the `Advanced-Hands-on` folder in the `$HOME/dcom-blr-handson` folder
2. inside the train folder [main.py](https://github.tools.sap/MLF/dcom-blr-handson/blob/main/Advanced-Hands-on/train/main.py) is the training script
3. requirements for the script are available in [requirements.txt](https://github.tools.sap/MLF/dcom-blr-handson/blob/main/Advanced-Hands-on/train/requirements.txt)
4. [Dockerfile](https://github.tools.sap/MLF/dcom-blr-handson/blob/main/Advanced-Hands-on/train/Dockerfile) has the steps needed to package the script and requirements into a container image
5. similarly serve folder has the serving script `main.py`, `requirements.txt` and a `Dockerfile`

Note: $HOME here refers to the home folder in your hands on laptop

### 3. Create and ship the docker images

Open Git bash program and start building the docker images

```bash
cd ~/dcom-blr-handson/Advanced-Hands-on/train

# check that docker is up and running. If you get an error when you run the below command
# Cannot connect to the Docker daemon at unix:///var/run/docker.sock. Is the docker daemon running?. make sure docker desktop is running
docker version

# Use the access token or password, winpty enables TTY login on windows for git bash
winpty docker login docker.io

# replace the placeholder below with your docker hub username
docker build -t docker.io/<YOUR_DOCKER_USERNAME>/house-price-train:01 .

docker push docker.io/<YOUR_DOCKER_USERNAME>/house-price-train:01


# create a serving image

cd ../serve

docker build -t docker.io/<YOUR_DOCKER_USERNAME>/house-price-server:01 .

docker push docker.io/<YOUR_DOCKER_USERNAME>/house-price-server:01

```

### 4. Onboard a git repository with workflow/serving templates

AI Launchpad URL - [https://dcom-blr-handson.ai-launchpad.prodintern.eu-central-1.aws.apps.ml.hana.ondemand.com](https://dcom-blr-handson.ai-launchpad.prodintern.eu-central-1.aws.apps.ml.hana.ondemand.com)

Click on the above link, you should be logged directly into the app. If prompted for credentials use the username and password details stored in the laptop you are accessing this hands on guide on (Home folder -> technical-users)

1. Fork this [repository](https://github.tools.sap/I521664/houseprice)
2. In the workflows folder, you will find 2 files. house-price-train.yaml and house-price-serv.yaml
3. Inside each file there is a placeholder `<your-name>`, replace the placeholder with your name or i number
4. Replace the placeholder `<your-docker-image>` with `docker.io/<YOUR_DOCKER_USERNAME>/house-price-train:01` in the `house-price-train.yaml` template
5. Replace the placeholder `<your-docker-image>` with `docker.io/<YOUR_DOCKER_USERNAME>/house-price-serve@<sha-hash>` in the `house-price-serve.yaml` template. When using docker hub it is necessary to provide the sha-hash when working with deployments. To fetch the image path run `docker inspect <image-id>`. From the response copy the image path from `RepoDigests` key. An example image path is `docker.io/sreei/house-price-server@sha256:a339e9f7202b91d63ea4dda10104c1608f56aed37f9ded099f3220aa100d67b2`
6. Replace the placeholder `<your-image-pull-secret>` with your `<i-number>-docker-registry-secret` in the `house-price-train.yaml` template
7. Replace the placeholder `<your-image-pull-secret>` with your `<i-number>-docker-registry-secret` in the `house-price-serve.yaml` template
8. In the Launchpad app, choose the AI API connection `dcom-handson`
9. Open the SAP AICORE Administration app and choose Git Repositories
10. Click on Add, in the dialog box add the following
    * URL: `<forked git url>`
    * Name: `<your-name>-repo`
    * User Name: `github.tools username`
    * Access Token: `github.tools access token`

Note: To Generate an access token, go to github settings. Inside the developer settings, create a new personal access token(with read permission).

### 5. Onboard an application
1. In the SAP AICORE Administration app choose Applications
2. Click on create, in the dialog box add the following
    * Application Name:  `<your-name>-app`
    * Repository: `Select the repo created earlier from the drop down`
    * Path in Repository: workflows
    * Revision: HEAD
3. Click on the created application and you can see the templates synced. 

Note: Click on sync button if you don't see the entry in the Sync details section

### 6. Create a docker registry secret
1. In the SAP AICORE Administration app choose Docker Registry Secrets
2. Click on Add, in the dialog box add the following
    * Name: `Name of the imagePullSecret specified in the workflow and serving templates.`
    * Secret: `json secret data`

    secret json format
    ```
    {
        ".dockerconfigjson": "{\"auths\":{\"docker.io\":{\"username\":\"john\",\"password\":\"docker-accesstoken\"}}}"
    }
    ```

Note: Replace username and access token values. Here the access token is the token you created in step 1.

### 7. Create a Resource Group
1. In the SAP AICORE Administration app choose Resource Groups
2. Click on Create, in the dialog box add the following
    * Resource Group ID: `dcom-<your-i-number>`

### 8. Register Object Store Secrets

Object Store is used as storage for your dataset, models, resultsets inside the SAP AI Core ecosystem. Currently AI Core supports S3, Azure blob storage, WebDHFS and Alicloud OSS. For our use case let's use S3.

1. In the Workspaces app, choose the AI API connection
2. Open the SAP AICORE Administration app and choose Object Store Secrets
3. Click on Add, in the dialog box select the resource group that is assigned to you
4. Add the following details in the object store
    * Resource Group: `Choose the Resource group assigned to you`
    * Name: default
    * Type: S3
    * Path Prefix: example-dataset/house-price-toy
    * Bucket: hcp-bdc928d9-c4e5-4295-a0ed-5b819b4a8f2c
    * Endpoint: s3-eu-central-1.amazonaws.com
    * Region: eu-central-1
    * Toogle `Use HTTPS` button on
    * Secret: 
            `{
            "AWS_ACCESS_KEY_ID": "AKIASRWQYWCKZBP6TQ6F",
            "AWS_SECRET_ACCESS_KEY": "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"
            }`


### 9. Register Training Dataset

1. Go back the Workspaces app, choose the AI API connection and select the resource group that is assigned to you
2. Open the ML Operations app and choose Datasets
3. Click on Add, in the Select Scenario page select the sceanrio
4. In the General Information page provide a name for the dataset, eg. House Price Dataset 101
5. In the URL page, provide the dataset URL as ai://default/data/jan
6. Review and add the dataset

Note:

If you break down the above dataset URL specification, we have

1. ai:// -> unique prefix
2. default -> object store secret name
3. data/jan -> folder in s3 where the dataset is located


### 10. Create Configuration

1. Inside the ML Operations app choose Configurations
2. Click on Create, in the Enter Name and Executable page add the following details
    * Configuration Name: training_configuration
    * Scenario: House Price (Tutorial)
    * Version: 1.0
    * Executable: training-metrics
3. In the Input Parameters page, add a value for the input parameter (DT_MAX_DEPTH) as 3
4. In the Input Artifacts page, from the right hand side select the registered dataset in the assignment column
5. Review and Create the configuration

### 11. Trigger Execution

1. Previous step lands us on configuration details page, from there click on create execution button to create the execution
2. The execution will initially be in Unknown state and after few mins, the execution goes to running state. Use the refresh button to check for the latest status.
3. Once the Execution reaches completed state, we can see that an output artifact (housepricemodel) is generated
4. We can also see some metrics associated with the execution


### 12. Create Serving Configuration

1. Inside the ML Operations app choose Configurations
2. Click on Create, in the Enter Name and Executable page add the following details
    * Configuration Name: serving_configuration
    * Scenario: House Price (Tutorial)
    * Version: 1.0
    * Executable: server
3. In the Input Parameters page, add a value for the input parameter (greetmessage) eg. hello
4. In the Input Artifacts page, from the right hand side select the generated model (housepricemodel) in the assignment column
5. Review and Create the configuration


### 13. Trigger Deployment

1. Previous step lands us on configuration details page, from there click on create deployment button
2. Keep the duration as standard, review and create the deployment
3. The deployment will initially be in Unknown state and after few mins (depends on the resource availability in the cluster), the deployment goes to running state. Use the refresh button to check for the latest status.
4. Once the deployment is Running we can see that a deployment URL is auto generated and is available on deployment details page.


Use any REST client to make prediction calls. Send a POST request to `<DEPLOYMENT_URL>/v2/predict` with your resource group as header `AI-Resource-Group: <resource-group>` and send the below payload to predict the price of a house when we use certain attributes as below

```
{
    "MedInc": 8.3252,
    "HouseAge": 41,
    "AveRooms": 6.984126984,
    "AveBedrms":1.023809524,
    "Population":322,
    "AveOccup": 2.555555556,
    "Latitude":37.88,
    "Longitude":-122.23
}

```

### Using curl

#### Token Generator

```
TOKEN=$(curl --silent --location "https://vscode5-xhd4bzz3.authentication.eu10.hana.ondemand.com/oauth/token?grant_type=client_credentials" --header "Authorization: Basic $AUTH" | jq -r .access_token)
```

Note: $AUTH is the authorization token which is saved in a file called auth_token in the laptop you are accessing this handson guide on (Home folder). We will set the auth token as environmental variable ($AUTH) for you, if in case the auth token is not available please set the environmental variable.

#### Inference call

```
curl --silent --location "https://api.ai.prod.eu-central-1.aws.ml.hana.ondemand.com/v2/inference/deployments/<deployment-id>/v2/predict" \
--header "Content-Type: application/json" \
--header "AI-Resource-Group: <resource-group-id>" \
--header "Authorization: Bearer $TOKEN" \
--data '{
    "MedInc": 8.3252,
    "HouseAge": 41,
    "AveRooms": 6.984126984,
    "AveBedrms":1.023809524,
    "Population":322,
    "AveOccup": 2.555555556,
    "Latitude":37.88,
    "Longitude":-122.23
}
'
```

Note: In the above inference call, replace the resource group and deployment id with your values

The prediction value you get is in hundreds of thousands of dollars ($100,000) for this use case.

That's it. Hope you found this hands on session informative. To know more about SAP AICORE and AI Launchpad use the below references.


### References

* [https://help.sap.com/docs/ai-launchpad](https://help.sap.com/docs/ai-launchpad)
* [https://help.sap.com/docs/ai-core](https://help.sap.com/docs/ai-core)
* [https://developers.sap.com/group.ai-core-get-started-basics.html](https://developers.sap.com/group.ai-core-get-started-basics.html)
* [https://pypi.org/project/ai-core-sdk/](https://pypi.org/project/ai-core-sdk/)
