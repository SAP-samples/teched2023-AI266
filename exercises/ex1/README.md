#  Introductory Hands on lab

# Use case
In this Hands on lab we will be training a [house price dataset](https://scikit-learn.org/stable/datasets/real_world.html#california-housing-dataset) available in scikit-learn. Then use the generated house price model to create a deployment and do online inferencing

# Introduction:

## AICORE:

SAP AI Core is a service designed to handle the execution and operations of your AI assets in a standardized, scalable, and hyperscaler-agnostic way. SAP AI Core supports full lifecycle management of your AI scenarios.

## AI Launchpad

SAP AI Launchpad is a multitenant software as a service (SaaS) application in SAP Business Technology Platform which is used to manage AI use cases (scenarios) across multiple instances of AI runtimes. AICORE is one such runtime.

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

In this hands on lab we will not go through all the above steps. Steps 1 to 4 are part of onboarding and we have done it for you.


<center><img src="./images/AICORE_workflow.png" alt="AICORE E2E workflow" style="width: 1000px;"/></center>


## Tenant and Resource Group Relationship

<left><img src="./images/Tenant_RG_Relationship.png" alt="Tenant Resource Group relationship" style="width: 800px;"/></left>

* Tenant is a higher level concept which corresponds to one AICORE subscription. 
* A Tenant can have many resource groups. 
* Resource groups are these isolated housing units for executions and deployments. 
* With this isolation one resource group cannot access the executions and deployments of another resource group.

# AIL Overview

There are 3 sub-applications to work with in the AI Launchpad app. 

1. Workspaces app
2. SAP AICORE Administration app
3. ML Operations app

### Workspaces App

Used to add AI API compliant runtimes to AI Launchpad app to manage the lifecycle of AI assets. The aicore service key (available after a subscription to AICORE service in BTP) is preconfigured in the AI Launchpad instance shared with you


### SAP AICORE Administration app

Used to perform administration activities for your SAP AI Core runtime

### ML Operations app

Used to trigger executions and deployments, watch the logs and visualize metrics after conducting AI experiments.

## Hands on steps

AI Launchpad URL - [https://dcom-blr-handson.ai-launchpad.prodintern.eu-central-1.aws.apps.ml.hana.ondemand.com](https://dcom-blr-handson.ai-launchpad.prodintern.eu-central-1.aws.apps.ml.hana.ondemand.com)

Click on the above link, you should be logged directly into the app. If prompted for credentials use the username and password details stored in the laptop you are accessing this hands on guide on (Home folder -> technical-users)

### 1. Register Object Store Secrets

Object Store is used as storage for your dataset, models, resultsets inside the SAP AI Core ecosystem. Currently AI Core supports S3, Azure blob storage, WebDHFS and Alicloud OSS. For our use case let's use S3.

1. In the Workspaces app, choose the AI API connection `dcom-handson`
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


### Register Training Dataset

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


### Create Configuration

1. Inside the ML Operations app choose Configurations
2. Click on Create, in the Enter Name and Executable page add the following details
    * Configuration Name: training_configuration
    * Scenario: House Price (Tutorial)
    * Version: 1.0
    * Executable: training-metrics
3. In the Input Parameters page, add a value for the input parameter (DT_MAX_DEPTH) as 3
4. In the Input Artifacts page, from the right hand side select the registered dataset in the assignment column
5. Review and Create the configuration

### Trigger Execution

1. Previous step lands us on configuration details page, from there click on create execution button to create the execution
2. The execution will initially be in Unknown state and after few mins, the execution goes to running state. Use the refresh button to check for the latest status.
3. Once the Execution reaches completed state, we can see that an output artifact (housepricemodel) is generated
4. We can also see some metrics associated with the execution


### Create Serving Configuration

1. Inside the ML Operations app choose Configurations
2. Click on Create, in the Enter Name and Executable page add the following details
    * Configuration Name: serving_configuration
    * Scenario: House Price (Tutorial)
    * Version: 1.0
    * Executable: server
3. In the Input Parameters page, add a value for the input parameter (greetmessage) eg. hello
4. In the Input Artifacts page, from the right hand side select the generated model (housepricemodel) in the assignment column
5. Review and Create the configuration


### Trigger Deployment

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
