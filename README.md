# Mushroom App: Serverless Model Training Demo

In this tutorial we will build a data pipeline flow as shown:
<img src="pipeline-flow.png"  width="800">

## Prerequisites
* Have Docker installed
* Cloned this repository to your local machine with a terminal up and running
* Check that your Docker is running with the following command

`docker run hello-world`

### Install Docker 
Install `Docker Desktop`

#### Ensure Docker Memory
- To make sure we can run multiple container go to Docker>Preferences>Resources and in "Memory" make sure you have selected > 4GB

### Install VSCode  
Follow the [instructions](https://code.visualstudio.com/download) for your operating system.  
If you already have a preferred text editor, skip this step.  

## Mushroom App: Setup Environments & Containers
In this tutorial we will setup a data labeling web app to label data for the mushroom app. We will use Docker to run everything inside containers.

**In order to complete this tutorial you will need your own GCP account setup and your github repo.**
### Clone the github repository
- Clone or download from [here](https://github.com/dlops-io/model-training)

## Setup GCP Credentials
Next step is to enable our container to have access to Storage buckets & Vertex AI(AI Platform) in  GCP. 

### Create a local **secrets** folder

It is important to note that we do not want any secure information in Git. So we will manage these files outside of the git folder. At the same level as the `model-training` folder create a folder called **secrets**

Your folder structure should look like this:
```
   |-model-training
   |-secrets
```

### Setup GCP Service Account
- Here are the step to create a service account:
- To setup a service account you will need to go to [GCP Console](https://console.cloud.google.com/home/dashboard), search for  "Service accounts" from the top search box. or go to: "IAM & Admins" > "Service accounts" from the top-left menu and create a new service account called "model-trainer". For "Service account permissions" select "Storage Admin", "AI Platform Admin", "Vertex AI Administrator".
- This will create a service account
- On the right "Actions" column click the vertical ... and select "Manage keys". A prompt for Create private key for "model-trainer" will appear select "JSON" and click create. This will download a Private key json file to your computer. Copy this json file into the **secrets** folder. Rename the json file to `model-trainer.json`


