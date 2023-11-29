# Edit Jumpstart Solution and deploy tutorial 

This tutorial provides the steps for you to build your own proof of concept solution based on the chosen Jumpstart Solution and deploy it. You can customize the chosen Jump start solutions (JSS) deployments by creating your own copy of the source code. You can modify the infrastructure and application code as needed and redeploy the solutions with the changes.

Each solution should be edited and deployed by one user at a time to avoid conflicts.

## Know your solution

Here are the details of the GenAI doc summarization Jump Start Solution chosen by you.

Solution Guide: [here](https://cloud.google.com/architecture/ai-ml/generative-ai-document-summarization)

The code for the solution is avaiable at the following location
* Infrastructure code directory is located under `./infra`
* Application code directory is located under `./source`


## Explore or Edit the solution as per your requirement

<Solution specific hints for the user to explore / modify>

Please note: to open your recently used workspace:
* Go to the `File` menu.
* Select `Open Recent Workspace`.
* Choose the desired workspace.


## Gather the required information for intializing gcloud command

In this step you will gather the information required for the deployment of the solution

**Project ID**

<walkthrough-project-setup></walkthrough-project-setup>

**Deployment Name**

Use the following command to list the deployments:
```bash
gcloud infra-manager deployments list --location=us-central1 --project=<walkthrough-project-id/>
```

```
Use above output to set the <var>DEPLOYMENT_NAME</var>
```

**User account email**

Use the following command to see the user account email:
```bash
gcloud config list | grep account
```
```
Use above output to set the <var>USER_EMAIL</var>
```


## Deploy the solution

**Set the gcloud config.**
```bash
gcloud config set account <var>USER_EMAIL</var>
gcloud config set project <walkthrough-project-id/>
```


**Fetch Deployment details**
```bash
gcloud infra-manager deployments describe projects/<walkthrough-project-id/>/locations/us-central1/deployments/<var>DEPLOYMENT_NAME</var>
```
From the output of this command, note down the input values provided in the existing deployment in the `terraformBlueprint.inputValues` section.

**Create the service account**
```bash
gcloud iam service-accounts create mim-journey
```

**Assign the required roles to the service account**
```bash
gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/aiplatform.admin"


gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/artifactregistry.reader"

gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/bigquery.admin"


gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/cloudfunctions.admin"


gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/eventarc.admin"


gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/iam.serviceAccountAdmin"


gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/iam.serviceAccountUser"


gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/logging.admin"


gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/pubsub.admin"


gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/resourcemanager.projectIamAdmin"


gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/run.admin"


gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/serviceusage.serviceUsageAdmin"


gcloud projects add-iam-policy-binding <walkthrough-project-id/> --member="serviceAccount:mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com" --role="roles/storage.admin"
```

**Create Terraform input file**

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

**Deploy the solution**

Execute the following command to trigger the re-deployment. 
```bash
gcloud infra-manager deployments apply projects/<walkthrough-project-id/>/locations/us-central1/deployments/<var>DEPLOYMENT_NAME</var> --service-account projects/<walkthrough-project-id/>/serviceAccounts/mim-journey@<walkthrough-project-id/>.iam.gserviceaccount.com --local-source="." --inputs-file=./input.tfvars --labels="modification-reason=make-it-mine,goog-solutions-console-deployment-name=generative-ai-document-summarization,goog-solutions-console-solution-id=generative-ai-document-summarization"
```

**Monitor the Deployment**

Execute the following command to get the deployment details.

```bash
gcloud infra-manager deployments describe - describe <var>DEPLOYMENT_NAME</var>
```

Monitor your deployment at [JSS deployment page](https://console.cloud.google.com/products/solutions/deployments?pageState=(%22deployments%22:(%22f%22:%22%255B%257B_22k_22_3A_22Labels_22_2C_22t_22_3A13_2C_22v_22_3A_22_5C_22modification-reason%2520_3A%2520make-it-mine_5C_22_22_2C_22s_22_3Atrue_2C_22i_22_3A_22deployment.labels_22%257D%255D%22))).

## Save your edits to the solution

Use any of the following methods to save your edits to the solution

**Download your solution in tar file**
* Click on the `File` menu
* Select `Download Workspace` to download the whole workspace on your local in compressed format.

**Save your solution to your git repository**

Set the remote url to your own repository
```bash 
git remote set-url origin [your-own-repo-url]
```

Review the modified files, commit and push to your remote repository branch.
