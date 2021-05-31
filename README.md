# GitLab CI to App Engine
Auto Deploy to Google App Engine from GitLab CI

## Requirements
- GitLab repository (Maintainer permission)
- Google Cloud PLatform project (Full permissions)

## Getting Started

### Create Service Account

Visit [iam-admin](https://console.cloud.google.com/iam-admin/serviceaccounts/create) to create a new Service Account. Add list of roles below:
 1. App Engine Admin
 2. Storage Object Admin
 3. Cloud Build Editor 
 4. Service Account User

After that, create and download the JSON key

### Config GitLab CI

In your Repo, go to Settings -> CICD then Explain **Variables** section, add 3 variables:

 1. `PROJECT_ID`: The ID of project
 2. `PROJECT_REGION`: Default project region
 3. `SERVICE_ACCOUT`: Paste the content of Service Account you have downloaded

*Note: Just need the type is `Variable`.*

### Config .gitlab-ci.yml

Create a new `.gitlab-ci.yml` if not:

```
image: google/cloud-sdk:alpine

deploy_production:
  stage: deploy
  environment: Production
  only:
  - master
  script:
  - echo $SERVICE_ACCOUNT > /tmp/$CI_PIPELINE_ID.json
  - gcloud auth activate-service-account --key-file /tmp/$CI_PIPELINE_ID.json
  - gcloud --quiet --project $PROJECT_ID app deploy app-production.yaml

deploy_staging:
  stage: deploy
  environment: Staging
  only:
  - staging
  script:
  - echo $SERVICE_ACCOUNT > /tmp/$CI_PIPELINE_ID.json
  - gcloud auth activate-service-account --key-file /tmp/$CI_PIPELINE_ID.json
  - gcloud --quiet --project $PROJECT_ID app deploy app-staging.yaml

deploy_development:
  stage: deploy
  environment: Development
  only:
  - cicd
  script:
  - echo $SERVICE_ACCOUNT > /tmp/$CI_PIPELINE_ID.json
  - gcloud auth activate-service-account --key-file /tmp/$CI_PIPELINE_ID.json
  - gcloud --quiet --project $PROJECT_ID app deploy app-development.yaml

after_script:
- rm /tmp/$CI_PIPELINE_ID.json
```

In this configuration, we will deploy `develop` branch to Development, `staging` branch to Staging and `master` branch to Production with different `app-<environment>.yaml` config for each environment.

*Note: Add `dispatch.yaml` to your deploy command if you need map routes*

