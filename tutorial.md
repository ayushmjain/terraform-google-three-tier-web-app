# Make it mine tutorial


## Introduction
---


* With Make it mine, customers can customize existing Jump start solutions (JSS) deployments by creating their own copy of the source code.
* They can then modify the infrastructure and application code as needed, and redeploy the solutions with their changes.
* Each solution should be edited and deployed by one person at a time to avoid conflicts.
* To open your recently used workspace:
    * Go to the `File` menu.
    * Select `Open Recent Workspace`.
    * Choose the desired workspace.

## Know your solution
---
**Pointers to modify the code**
* Infrastructure code directory is located under `./infra`
* Application code directory is located under `./source`
---
**Documentation**
* Find the documentation related to the solution [here](https://github.com/GoogleCloudPlatform/terraform-genai-doc-summarization).

## Gathering required information
---
**Project ID**
```bash
read -p "Please enter your project id: " PROJECT_ID
```
---
**Deployment Name Discovery**
```bash
read -p "Please enter your deployment name: " DEPLOYMENT_NAME
```
---
**User account email**

```bash
read -p "Please enter your email: " USER_EMAIL
```

## Deployment Steps
---
**Set the gcloud config.**
```bash
gcloud config set account $USER_EMAIL
gcloud config set project $PROJECT_ID
```

---
**Fetch the existing deployment**
```bash
gcloud infra-manager deployments describe projects/$PROJECT_ID/locations/us-central1/deployments/generative-ai-document-summarization
```
From the output of this command, note down the input values provided in the existing deployment in the `terraformBlueprint.inputValues` section.

---
**Create the service account**
```bash
gcloud iam service-accounts create mim-journey
```

---
**Assign the required roles to the service account**
```bash
gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/aiplatform.admin"


gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/artifactregistry.reader"

gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/bigquery.admin"


gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/cloudfunctions.admin"


gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/eventarc.admin"


gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/iam.serviceAccountAdmin"


gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/iam.serviceAccountUser"


gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/logging.admin"


gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/pubsub.admin"


gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/resourcemanager.projectIamAdmin"


gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/run.admin"


gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/serviceusage.serviceUsageAdmin"


gcloud projects add-iam-policy-binding <gcp-project> --member="serviceAccount:mim-journey@<gcp-project>.iam.gserviceaccount.com" --role="roles/storage.admin"
```

---
**Terraform Inputs**

Create `input.tfvars` file.

Find the sample content below and modify it by providing the respective details.
```
delete_contents_on_destroy = true
deployment_name = <input_your_deployment_name_here>
project_id = <input_your_project_name_here>
region = "us-central1"
labels = {
    "goog-solutions-console-deployment-name" = "generative-ai-document-summarization",
    "goog-solutions-console-solution-id" = "generative-ai-document-summarization"
}
```

---
**Deployment**

Execute the following command to trigger the re-deployment.
```bash
gcloud infra-manager deployments apply projects/$PROJECT_ID/locations/us-central1/deployments/$DEPLOYMENT_NAME --service-account projects/$PROJECT_ID/serviceAccounts/mim-journey@$PROJECT_ID.iam.gserviceaccount.com --local-source="."     --inputs-file=./input.tfvars --labels="modification-reason=make-it-mine,goog-solutions-console-deployment-name=generative-ai-document-summarization,goog-solutions-console-solution-id=generative-ai-document-summarization"
```

---
**Deployment Status Monitoring**

Execute the following command to get the deployment details.

```bash
gcloud infra-manager deployments describe - describe $DEPLOYMENT_NAME
```

---
**Deployment page**

The make it mine deployment can be found on the [JSS deployment page](https://console.cloud.google.com/products/solutions/deployments?pageState=(%22deployments%22:(%22f%22:%22%255B%257B_22k_22_3A_22Labels_22_2C_22t_22_3A13_2C_22v_22_3A_22_5C_22modification-reason%2520_3A%2520make-it-mine_5C_22_22_2C_22s_22_3Atrue_2C_22i_22_3A_22deployment.labels_22%257D%255D%22))).

## Save your work
Use any of the below method to save your modified code.

---
**Download the tar**
* Click on the `File` menu
* Select `Download Workspace` to download the whole workspace on your local in compressed format.

---
**Setup remote repo**

Set the remote url to your own repository
```bash 
git remote set-url origin [your-own-repo-url]
```

Review the modified files, commit and push to your remote repository branch.
