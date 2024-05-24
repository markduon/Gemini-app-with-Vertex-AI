# Develop an App with Vertex AI Gemini 1.0 Pro

## Configure your environment and project

1. Sign in to the Google Cloud console with your lab credentials, and open the Cloud Shell terminal window.
2. To set your project ID and region environment variables, in Cloud Shell, run the following commands:
```
PROJECT_ID=$(gcloud config get-value project)
REGION=us-east4
echo "PROJECT_ID=${PROJECT_ID}"
echo "REGION=${REGION}"
```
3. In order to use various Google Cloud services in this lab, you must enable a few APIs:
```
gcloud services enable cloudbuild.googleapis.com cloudfunctions.googleapis.com run.googleapis.com logging.googleapis.com storage-component.googleapis.com aiplatform.googleapis.com
```

## Run and test the app locally

1. To run the app locally, in Cloud Shell, execute the command:
```
streamlit run app.py \
--browser.serverAddress=localhost \
--server.enableCORS=false \
--server.enableXsrfProtection=false \
--server.port 8080
```
2. To launch the app home page in your browser, click web preview in the Cloud Shell menubar, and then click Preview on port 8080.

## Deploy the app to Cloud Run

### Set up the environment:

1. Make sure you are in the app directory:
```
cd ~/gemini-app
```

2. Verify that the PROJECT_ID, and REGION environment variables are set:
```
echo "PROJECT_ID=${PROJECT_ID}"
echo "REGION=${REGION}"
```

3. If these environment variables are not set, then run the command to set them:
```
PROJECT_ID=$(gcloud config get-value project)
REGION=us-east4
echo "PROJECT_ID=${PROJECT_ID}"
echo "REGION=${REGION}"
```

4. Set environment variables for your service and artifact repository:
```
SERVICE_NAME='gemini-app-playground' # Name of your Cloud Run service.
AR_REPO='gemini-app-repo'            # Name of your repository in Artifact Registry that stores your application container image.
echo "SERVICE_NAME=${SERVICE_NAME}"
echo "AR_REPO=${AR_REPO}"
```

### Create the Docker repository

1. To create the repository in Artifact Registry, run the command:
```
gcloud artifacts repositories create "$AR_REPO" --location="$REGION" --repository-format=Docker
```

2. Set up authentication to the repository:
```
gcloud auth configure-docker "$REGION-docker.pkg.dev"
```

### Build the container image

1. To build the container image for your app, run the command:
```
gcloud builds submit --tag "$REGION-docker.pkg.dev/$PROJECT_ID/$AR_REPO/$SERVICE_NAME"
```

### Deploy and test your app on Cloud Run

The final task is to deploy the service to Cloud Run with the image that was built and pushed to the repository in Artifact Registry.

1. To deploy your app to Cloud Run, run the command:
```
gcloud run deploy "$SERVICE_NAME" \
  --port=8080 \
  --image="$REGION-docker.pkg.dev/$PROJECT_ID/$AR_REPO/$SERVICE_NAME" \
  --allow-unauthenticated \
  --region=$REGION \
  --platform=managed  \
  --project=$PROJECT_ID \
  --set-env-vars=PROJECT_ID=$PROJECT_ID,REGION=$REGION
```

2. After the service is deployed, a URL to the service is generated in the output of the previous command. To test your app on Cloud Run, navigate to that URL in a separate browser tab or window.

3. Choose the app functionality that you want to test. The app will prompt the Vertex AI Gemini API to generate and display the responses.