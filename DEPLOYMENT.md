# Deployment Guide

This guide explains how to deploy **AI Code Inspector** to Google Cloud Run and configure GitHub Webhooks for automated PR reviews.

## Prerequisites

- Google Cloud Project with billing enabled
- `gcloud` CLI installed and authenticated
- GitHub Repository
- Gemini API Key
- GitHub Personal Access Token (Repo scope)

## 1. Environment Setup

Set your environment variables:

```bash
export PROJECT_ID="your-project-id"
export REGION="us-central1"
export IMAGE_NAME="codeinspector"
export GOOGLE_API_KEY="your-gemini-key"
export GITHUB_TOKEN="your-github-token"
export GITHUB_WEBHOOK_SECRET="your-chosen-secret"
```

## 2. Build and Push Docker Image

Submit the build to Cloud Build (easiest method):

```bash
gcloud builds submit --tag gcr.io/$PROJECT_ID/$IMAGE_NAME
```

Or build locally and push:
```bash
docker build -t gcr.io/$PROJECT_ID/$IMAGE_NAME .
docker push gcr.io/$PROJECT_ID/$IMAGE_NAME
```

## 3. Deploy to Cloud Run

Deploy the service, passing the environment variables:

```bash
# Check if variables are set
if [ -z "$PROJECT_ID" ] || [ -z "$REGION" ]; then
  echo "Error: Please set PROJECT_ID and REGION variables."
  exit 1
fi

gcloud run deploy codeinspector-service \
  --image gcr.io/$PROJECT_ID/$IMAGE_NAME \
  --platform managed \
  --region $REGION \
  --allow-unauthenticated \
  --set-env-vars GOOGLE_API_KEY=$GOOGLE_API_KEY,GITHUB_TOKEN=$GITHUB_TOKEN,GITHUB_WEBHOOK_SECRET=$GITHUB_WEBHOOK_SECRET
```

After deployment, note the **Service URL** (e.g., `https://codeinspector-service-xyz-uc.a.run.app`).

## 4. Configure GitHub Webhook

1.  Go to your GitHub Repository > **Settings** > **Webhooks**.
2.  Click **Add webhook**.
3.  **Payload URL**: `https://<YOUR-SERVICE-URL>/webhook/github`
4.  **Content type**: `application/json`
5.  **Secret**: The `GITHUB_WEBHOOK_SECRET` you set in step 3.
6.  **Which events would you like to trigger this webhook?**: Select **Let me select individual events** and check **Pull requests**.
7.  Click **Add webhook**.

## 5. Verification

1.  Create a new Pull Request in your repository.
2.  Check the "Recent Deliveries" in the GitHub Webhook settings to see the request sent to Cloud Run.
3.  Check Cloud Run logs to see the processing.
4.  The AI Code Inspector should comment on your PR automatically!
