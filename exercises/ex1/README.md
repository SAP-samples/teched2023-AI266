#  SAP AICORE and AI Launchpad Hands on lab

# Use case
In this Hands on lab we will be training a [house price](https://scikit-learn.org/stable/datasets/real_world.html#california-housing-dataset) dataset. Then use the generated house price model to create a deployment and do online inferencing

# Introduction:

## AICORE:

SAP AI Core is a service designed to handle the execution and operations of your AI assets in a standardized, scalable, and hyperscaler-agnostic way. SAP AI Core supports full lifecycle management of your AI scenarios.

## AI Launchpad

SAP AI Launchpad is a multitenant software as a service (SaaS) application in SAP Business Technology Platform which is used to manage AI workflows across multiple AI runtimes. AICORE is one such runtime.

## AICORE End to End Workflow

1. Register your Object Store
2. Register a Training Dataset
3. Create a Configuration for your Training Executable
4. Training
5. Create a Configuration for your Serving Executable
6. Serve the model as a web service

**_Disclaimer: Please note, certain steps pertaining to the onboarding process have been pre-emptively completed for your convenience. For a more detailed understanding of the onboarding process, please refer to our [help guide](https://help.sap.com/docs/sap-ai-core/sap-ai-core-service-guide/initial-setup)_**


<center><img src="./images/AICORE_workflow.png" alt="AICORE E2E workflow" style="width: 1000px;"/></center>


## Tenant and Resource Group Relationship

<left><img src="./images/Tenant_RG_Relationship.png" alt="Tenant Resource Group relationship" style="width: 800px;"/></left>

* Tenant is a higher level construct which corresponds to one AICORE subscription. 
* A Tenant can have many resource groups. 
* Resource groups are isolated housing units for your executions and deployments. 
* With this isolation, one resource group cannot access the executions and deployments of another resource group.

# AI Launchpad Overview

There are 3 sub-applications to work with in AI Launchpad. 

1. Workspaces app
2. SAP AICORE Administration app
3. ML Operations app

### Workspaces App

Workspaces app is used to configure AI API compliant runtimes to AI Launchpad as connections. The aicore service key (available after a subscription to AICORE service in BTP) is preconfigured in the AI Launchpad instance shared with you.


### SAP AICORE Administration app

Used to perform administration activities for your SAP AI Core runtime

### ML Operations app

Used to trigger executions and deployments, watch the logs and visualize metrics after conducting AI experiments.

# Hands on steps

AI Launchpad URL - [https://teched-launchpad-demo.ai-launchpad.prodintern.eu-central-1.aws.apps.ml.hana.ondemand.com](https://teched-launchpad-demo.ai-launchpad.prodintern.eu-central-1.aws.apps.ml.hana.ondemand.com)

Click on the above link, you should be logged directly into the app. If prompted for credentials provide the username as `DL_652F6D2C1B59E6028C215346@global.corp.sap` and for password please check with the presenters.

### 1. Register Object Store

Object Store is used as storage for your datasets, models, resultsets inside the SAP AI Core ecosystem. Currently AI Core supports S3, Azure blob storage, WebDHFS and Alicloud OSS. For our use case let's use S3.

1. In the Workspaces app, choose the AI API connection `teched-handson`

<left><img src="./images/Workspaces.png" alt="Workspaces App" style="width: 800px;"/></left>

2. Open the SAP AICORE Administration app and choose Object Store Secrets
3. On the right top hand corner select the resource group that is assigned to you (if you're not sure reach out to the presenters)
4. Click on Add,
5. Add the following details in the secret form
    * Resource Group: `Choose the Resource group assigned to you`
    * Name: default
    * Type: S3
    * Path Prefix: example-dataset/house-price-toy
    * Bucket: hcp-3125e8fa-d1b6-4409-ba0c-a52d400ecfab
    * Endpoint: s3-eu-central-1.amazonaws.com
    * Region: eu-central-1
    * Toogle `Use HTTPS` button on
    * Secret: 
            `{
            "AWS_ACCESS_KEY_ID": "AKIAWSCM7XP3OLAXE7E6",
            "AWS_SECRET_ACCESS_KEY": "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"
            }`

**Note: Please ensure to use the keep the secret name as `default`**

<left><img src="./images/Object_Store.png" alt="Object Store Secrets Page" style="width: 800px;"/></left>

### 2. Register Training Dataset

1. Go back the Workspaces app, choose the AI API connection `teched-handson` and select the resource group that is assigned to you
2. Now open the ML Operations app and choose Datasets
3. Click on Add, in the Select Scenario page select the sceanrio `House Price (Tutorial)`

<left><img src="./images/Datasets_Scenario.png" alt="Datasets Scenario Page" style="width: 800px;"/></left>

4. In the General Information page provide a name for the dataset, eg. House Price Dataset 101
5. In the URL page, provide the dataset URL as `ai://default/data/jan`

<left><img src="./images/Datasets_URL.png" alt="Datasets URL Page" style="width: 800px;"/></left>

6. Review and add the dataset

Note:

If you break down the above dataset URL specification, we have

1. ai:// -> prefix
2. default -> object store secret name
3. data/jan -> folder in s3 where the dataset is located


### 3. Create Configuration

1. Inside the ML Operations app choose Configurations
2. Click on Create, in the Enter Name and Executable page add the following details
    * Configuration Name: training_configuration
    * Scenario: House Price (Tutorial)
    * Version: 1.0
    * Executable: training-metrics

<left><img src="./images/Configurations_Executable.png" alt="Configurations Name and Executable Page" style="width: 800px;"/></left>

3. In the Input Parameters page, add a value for the input parameter (DT_MAX_DEPTH) as 3
4. In the Input Artifacts page, from the right hand side select the registered dataset in the assignment column

<left><img src="./images/Configurations_Artifact.png" alt="Configurations Input Artifacts Page" style="width: 800px;"/></left>

5. Review and Create the configuration

### 4. Trigger Execution

1. Previous step lands us on configuration details page, from there click on create execution button to create the execution

<left><img src="./images/Configuration_Details.png" alt="Configuration Details Page" style="width: 800px;"/></left>

2. The execution will initially be in Unknown state and after few mins, the execution goes to running state. Use the refresh button to check for the latest status.

Note: Based on the load on the system, it usually takes some time for the execution to go to Running state.

<left><img src="./images/Execution_Running.png" alt="Execution Running" style="width: 800px;"/></left>

3. Once the Execution reaches completed state, we can see that an output artifact (housepricemodel) is generated

<left><img src="./images/Execution_Completed.png" alt="Execution Completed" style="width: 800px;"/></left>

4. We can also see some metrics associated with the execution

<left><img src="./images/Metrics.png" alt="Metrics" style="width: 800px;"/></left>

### 5. Create Serving Configuration

1. Inside the ML Operations app choose Configurations
2. Click on Create, in the Enter Name and Executable page add the following details
    * Configuration Name: serving_configuration
    * Scenario: House Price (Tutorial)
    * Version: 1.0
    * Executable: server

<left><img src="./images/Configurations_Serving_Executable.png" alt="Configurations Serving Name and Executable Page" style="width: 800px;"/></left>

3. In the Input Parameters page, add a value for the input parameter (greetmessage) eg. hello
4. In the Input Artifacts page, from the right hand side select the generated model (housepricemodel) in the assignment column

<left><img src="./images/Configurations_Serving_Artifact.png" alt="Configurations Serving Artifacts Page" style="width: 800px;"/></left>

5. Review and Create the configuration


### 6. Trigger Deployment

1. Previous step lands us on configuration details page, from there click on create deployment button

<left><img src="./images/Create_Deployment.png" alt="Create Deployment" style="width: 800px;"/></left>

2. Keep the duration as standard, review and create the deployment
3. The deployment will initially be in Unknown state and after few minutes the deployment goes to running state. Use the refresh button to check for the latest status.

    Note: Based on the load on the system, it usually takes some time for the deployment to go to Running state.

4. Once the deployment is Running we can see that a deployment URL is generated.

<left><img src="./images/Deployment_Running.png" alt="Deployment Running" style="width: 800px;"/></left>


## Online Inferencing

Now that a deployment URL is available we can use a REST client to make inference calls. Send a POST request to `<DEPLOYMENT_URL>/v2/predict` with your resource group as header `AI-Resource-Group: <resource-group>` and send the below payload to predict the price of a house

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
TOKEN=$(curl --silent --location "https://teched-launchpad-demo.authentication.sap.hana.ondemand.com/oauth/token?grant_type=client_credentials" --header "Authorization: Basic c2ItODNjOTVkZTYtZjRmOS00ODMyLWJmZmYtZTBiNWUwYWRiOTZlIWI5Nzg5N3x4c3VhYV9zdGQhYjc3MDg5OmI2Y2RhNTEyLTU0OTYtNDYzMy05NTg5LTVmZmNmYWZlMmYxNyQ2Q3U3enplNElGTDZTeU5rdE1UdXc5eVc3MlZVLXZJTGUxSEpQem14UW1ZPQ==" | jq -r .access_token)
```

#### Inference call

```
curl --silent --location "https://api.ai.internalprod.eu-central-1.aws.ml.hana.ondemand.com/v2/inference/deployments/<your-deployment-id>/v2/predict" \
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

Note: In the above inference call, replace the resource group and deployment id with your values.

The prediction value you get is in hundreds of thousands of dollars ($100,000).

### Conclusion

Congratulations on successfully training and deploying a machine learning model.

Utilizing SAP AI Core and AI Launchpad, you have accomplished AI use-case realization in a highly standardized, scalable, and hyperscaler-agnostic manner. This hands-on session provided valuable insights into managing AI assets, training models on datasets, generating machine learning models, and ultimately deploying these models for online inference.

Hope you found this session informative. To know more about SAP AICORE and AI Launchpad refer to the ensuing documents.


### References

* [https://help.sap.com/docs/ai-launchpad](https://help.sap.com/docs/ai-launchpad)
* [https://help.sap.com/docs/ai-core](https://help.sap.com/docs/ai-core)
* [https://developers.sap.com/group.ai-core-get-started-basics.html](https://developers.sap.com/group.ai-core-get-started-basics.html)
* [https://pypi.org/project/ai-core-sdk/](https://pypi.org/project/ai-core-sdk/)
