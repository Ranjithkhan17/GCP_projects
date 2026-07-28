# Build Serverless Applications with Cloud Run Functions: Challenge Lab


Set the project code
```
gcloud config set project PROJECT_ID
```
Set Variables:
```
export PROJECT_ID=$(gcloud config get-value project)
export REGION=LOCATION(AS PER TASK)
export BUCKET_NAME=$PROJECT_ID
PROJECT_NUMBER=$(gcloud projects describe $PROJECT_ID --format='value(projectNumber)')
```
Enable APIs:
```
gcloud services enable \
eventarc.googleapis.com \
pubsub.googleapis.com \
run.googleapis.com \
storage.googleapis.com
```
Create Bucket:
```
gcloud storage buckets create UNIQUE_BUCKET_NAME \
--location=$REGION
```

Give eventarc role to service account:
```

gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member=serviceAccount:$PROJECT_NUMBER-compute@developer.gserviceaccount.com \
    --role=roles/eventarc.eventReceiver
```
	------------------------------------------------------------
Create and configure the Cloud Storage service agent:
```
	
	SERVICE_ACCOUNT="$(gcloud storage service-agent --project=$PROJECT_ID)"
	
	 gcloud projects add-iam-policy-binding $PROJECT_ID \
  --member="serviceAccount:${GCS_SERVICE_ACCOUNT}" \
  --role="roles/pubsub.publisher"
  ```
 Give pubsub role to service account:

```
gcloud projects add-iam-policy-binding $PROJECT_ID \
    --member="serviceAccount:PROJECT_NUMBER-compute@developer.gserviceaccount.com" \
    --role='roles/pubsub.publisher'
```

Create a directory and inside the directory, create index.js and package.json

```
mkdir cs-logger #example
cd cs-loogger
```

index.js
```
const functions = require('@google-cloud/functions-framework');

functions.cloudEvent('eventStorage', (cloudevent) => {
  console.log('A new event in your Cloud Storage bucket has been logged!');
  console.log(cloudevent);
});
```
package.json
```
{
  "name": "nodejs-functions-gen2-codelab",
  "version": "0.0.1",
  "main": "index.js",
  "dependencies": {
    "@google-cloud/functions-framework": "^2.0.0"
  }
}
```

Create, deploy, and test a Cloud Storage function
```
	gcloud functions deploy FUNCTION_NAME \--gen2 \
  --runtime=nodejs24 \
  --region=REGION \
  --source=. \
  --entry-point=FUNCTION_NAME \
  --trigger-bucket=BUCKET_NAME\
  --max-instances=2
  ```
========================================================

Create and deploy a HTTP function with minimum instances

Create a directory and create index.js and package.json file inside it.

```
	gcloud functions deploy FUNCTION_NAME \--gen2 \
  --runtime=nodejs24 \
  --region=REGION \
  --source=. \
  --entry-point=FUNCTION_NAME \
  --trigger-bucket=BUCKET_NAME\
  --max-instances=2
```


*************************CONGRATULATIONS*************************


