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

## Setup Environments
In this tutorial we will setup a container to manage packaging python code for training and creating jobs on Vertex AI (AI Platform) to run training tasks.

**In order to complete this tutorial you will need your GCP account setup and a WandB account setup.**

### Clone the github repository
- Clone or download from [here](https://github.com/dlops-io/model-training)

### API's to enable in GCP for Project
Search for each of these in the GCP search bar and click enable to enable these API's
* Vertex AI API
* Cloud Storage API

### Setup GCP Credentials
Next step is to enable our container to have access to Storage buckets & Vertex AI(AI Platform) in  GCP. 

#### Create a local **secrets** folder

It is important to note that we do not want any secure information in Git. So we will manage these files outside of the git folder. At the same level as the `model-training` folder create a folder called **secrets**

Your folder structure should look like this:
```
   |-model-training
   |-secrets
```

#### Setup GCP Service Account
- Here are the step to create a service account:
- To setup a service account you will need to go to [GCP Console](https://console.cloud.google.com/home/dashboard), search for  "Service accounts" from the top search box. or go to: "IAM & Admins" > "Service accounts" from the top-left menu and create a new service account called "model-trainer". For "Service account permissions" select "Storage Admin", "AI Platform Admin", "Vertex AI Administrator".
- This will create a service account
- On the right "Actions" column click the vertical ... and select "Manage keys". A prompt for Create private key for "model-trainer" will appear select "JSON" and click create. This will download a Private key json file to your computer. Copy this json file into the **secrets** folder. Rename the json file to `model-trainer.json`

### Create GCS Bucket

We need a bucket to store the packaged python files that we will use for training.

- Go to `https://console.cloud.google.com/storage/browser`
- Create a bucket `mushroom-app-trainer` [REPLACE WITH YOUR BUCKET NAME]

### Get WandB Account API Key

We want to track our model training runs using WandB. Get the API Key for WandB: 
- Login into [WandB](https://wandb.ai/home)
- Go to to [User settings](https://wandb.ai/settings)
- Scroll down to the `API keys` sections 
- Copy the key
- Set an environment variable using your terminal: `export WANDB_KEY=...`
<img src="wandb-api-key.png"  width="400">

## Run Container

### Run `docker-shell.sh` or `docker-shell.bat`
Based on your OS, run the startup script to make building & running the container easy

This is what your `docker-shell` file will look like:
```
export IMAGE_NAME=model-training-cli
export BASE_DIR=$(pwd)
export SECRETS_DIR=$(pwd)/../secrets/
export GCS_BUCKET_URI="gs://mushroom-app-trainer" [REPLACE WITH YOUR BUCKET NAME]
export GCP_PROJECT="mlproject01-207413" [REPLACE WITH YOUR PROJECT]
export GCP_REGION="us-central1"


# Build the image based on the Dockerfile
docker build -t $IMAGE_NAME -f Dockerfile .
# M1/2 chip macs use this line
#docker build -t $IMAGE_NAME --platform=linux/arm64/v8 -f Dockerfile .

# Run Container
docker run --rm --name $IMAGE_NAME -ti \
-v "$BASE_DIR":/app \
-v "$SECRETS_DIR":/secrets \
-e GOOGLE_APPLICATION_CREDENTIALS=/secrets/model-trainer.json \
-e GCP_PROJECT=$GCP_PROJECT \
-e GCP_REGION=$GCP_REGION \
-e GCS_BUCKET_URI=$GCS_BUCKET_URI \
-e WANDB_KEY=$WANDB_KEY \ [MAKE SURE YOU HAVE THIS VARIABLE SET]
$IMAGE_NAME
```

- Make sure you are inside the `model-training` folder and open a terminal at this location
- Run `sh docker-shell.sh` or `docker-shell.bat` for windows
- The `docker-shell` file assumes you have the `WANDB_KEY` as an environment variable and is passed into the container


### Package & Upload Python Code

### Review Trainer Code
- Open & Review `model-training` > `setup.py`
- All required third party libraries needed for training are specified in `setup.py`
- Open & Review `model-training` > `package` > `trainer` > `task.py`
- All training code for the mushroom app models are present in `task.py`

### Run `sh package-trainer.sh`
- This script will create a `trainer.tar.gz` file with all the training code bundled inside it
- Then this script will upload this packaged file to your GCS bucket can call it `mushroom-app-trainer.tar.gz`


### Create Jobs in Vertex AI
- Open & Review `model-training` > `cli.sh`
- `cli.sh` is a script file to make calling `gcloud ai custom-jobs create` easier by maintaining all the parameters in the script
- Run `sh cli.sh`

### View Jobs in Vertex AI
- Go to Vertex AI [Custom Jobs](https://console.cloud.google.com/vertex-ai/training/custom-jobs)
- You will see the newly created job ready to be provisioned to run. 

### View Training Metrics
- Go to [WandB](https://wandb.a)
- Select the project `mushroom-training-vertex-ai`
- You will view the training metrics tracked and automatically updated