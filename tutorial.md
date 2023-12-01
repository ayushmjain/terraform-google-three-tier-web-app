<walkthrough-metadata>
  <meta name="title" content="Edit Jumpstart Solution and deploy tutorial " />
   <meta name="description" content="Make it mine neos tutorial" />
  <meta name="component_id" content="1361081" />
  <meta name="unlisted" content="true" />
  <meta name="short_id" content="true" />
</walkthrough-metadata>

# Customize Three-tier web app Solution

This tutorial provides the steps for you to build your own proof of concept solution based on the chosen Jumpstart Solution and deploy it. You can customize the chosen Jump start solutions (JSS) deployments by creating your own copy of the source code. You can modify the infrastructure and application code as needed and redeploy the solutions with the changes.

Each solution should be edited and deployed by one user at a time to avoid conflicts.

## Know your solution

Here are the details of the Three-tier web app Jump Start Solution chosen by you.

Solution Guide: [here](https://cloud.google.com/architecture/application-development/three-tier-web-app)

The code for the solution is avaiable at the following location
* Infrastructure code is present as part of `./main.tf`
* Application code directory is located under `./src`


## Explore or Edit the solution as per your requirement

The application source code for the frontend service is present under `src/frontend` directory and for the middleware service under `src/middleware` directory. 

Both these services are built as docker images and deployed using cloud run. The IaC code / terraform code is present in the `*.tf` files in the current directory.

NOTE: The changes in infrastructure may lead to reduction or increase in the incurred cost.

Please note: to open your recently used workspace:
* Go to the `File` menu.
* Select `Open Recent Workspace`.
* Choose the desired workspace.


---
**Automated deployment**

Execute the below command if you want to an automated deployment to happen without following the full tutorial.

The step is optional and you can continue with the full tutorial if you want to understand the individual steps involved in the script.

```bash
./deploy.sh
```

## Gather the required information for intializing gcloud command

In this step you will gather the information required for the deployment of the solution

---
**Project ID**

Use the following command to see the projectId:

```bash
gcloud config get project
```

```
Use above output to set the <var>PROJECT_ID</var>
```

---
**Deployment Name**

Use the following command to list the deployments:
```bash
gcloud infra-manager deployments list --location us-central1 --filter="labels.goog-solutions-console-deployment-name:*"
```

```
Use above output to set the <var>DEPLOYMENT_NAME</var>
```


## Deploy the solution


---
**Fetch Deployment details**
```bash
gcloud infra-manager deployments describe <var>DEPLOYMENT_NAME</var> --location us-central1
```
From the output of this command, note down the input values provided in the existing deployment in the `terraformBlueprint.inputValues` section.

Also note the serviceAccount from the output of this command. The value of this field is of the form 
```
projects/<var>PROJECT_ID</var>/serviceAccounts/<service-account>@<var>PROJECT_ID</var>.iam.gserviceaccount.com
```

```
Note <service-account> part and set the <var>SERVICE_ACCOUNT</var> value.
You can also set it to any exising service account.
```

----
**Create docker images**

NOTE: Modify the Image tags incrementally. Sample value="1.0.0"

Execute the following command to build and push the docker image for middleware and frontend:
```bash
cd ./src/middleware
gcloud builds submit --config=./cloudbuild.yaml --substitutions=_IMAGE_TAG="<var>IMAGE_TAG</var>"
cd -
cd ./src/frontend
gcloud builds submit --config=./cloudbuild.yaml --substitutions=_IMAGE_TAG="<var>IMAGE_TAG</var>"
cd -
```

Modify the `api_image` and `fe_image` value in main.tf with the updated Image tag.
```
locals {
  api_image = "gcr.io/<var>PROJECT_ID</var>/three-tier-app-be:<var>IMAGE_TAG</var> "
  fe_image  = "gcr.io/<var>PROJECT_ID</var>/three-tier-app-fe:<var>IMAGE_TAG</var> "
}
```

---
**Assign the required roles to the service account**
```bash
gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" --role="roles/artifactregistry.admin"


gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" --role="roles/cloudsql.admin"


gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" --role="roles/compute.networkAdmin"


gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" --role="roles/iam.serviceAccountAdmin"


gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" --role="roles/iam.serviceAccountUser"


gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" --role="roles/redis.admin"


gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" --role="roles/resourcemanager.projectIamAdmin"


gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" --role="roles/run.admin"


gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" --role="roles/servicenetworking.serviceAgent"


gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" --role="roles/serviceusage.serviceUsageAdmin"


gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" --role="roles/serviceusage.serviceUsageViewer"


gcloud projects add-iam-policy-binding <var>PROJECT_ID</var> --member="serviceAccount:<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com" --role="roles/vpcaccess.admin"
```

---
**Create Terraform input file**

Create `input.tfvars` file.

Find the sample content below and modify it by providing the respective details.
```
delete_contents_on_destroy=True
region="us-central1"
zone="us-central1-b"
project_id = "<var>PROJECT_ID</var>"
deployment_name = "<var>DEPLOYMENT_NAME</var>"
labels = {
  "goog-solutions-console-deployment-name" = "<var>DEPLOYMENT_NAME</var>",
  "goog-solutions-console-solution-id" = "three-tier-web-app"
}
```

---
**Deploy the solution**

Execute the following command to trigger the re-deployment. 
```bash
gcloud infra-manager deployments apply projects/<var>PROJECT_ID</var>/locations/us-central1/deployments/<var>DEPLOYMENT_NAME</var> --service-account projects/<var>PROJECT_ID</var>/serviceAccounts/<var>SERVICE_ACCOUNT</var>@<var>PROJECT_ID</var>.iam.gserviceaccount.com --local-source="."     --inputs-file=./input.tfvars --labels="modification-reason=make-it-mine,goog-solutions-console-deployment-name=<var>DEPLOYMENT_NAME</var>,goog-solutions-console-solution-id=three-tier-web-app"
```

---
**Monitor the Deployment**

Execute the following command to get the deployment details.

```bash
gcloud infra-manager deployments describe <var>DEPLOYMENT_NAME</var> --location us-central1
```

Monitor your deployment at [JSS deployment page](https://console.cloud.google.com/products/solutions/deployments?pageState=(%22deployments%22:(%22f%22:%22%255B%257B_22k_22_3A_22Labels_22_2C_22t_22_3A13_2C_22v_22_3A_22_5C_22modification-reason%2520_3A%2520make-it-mine_5C_22_22_2C_22s_22_3Atrue_2C_22i_22_3A_22deployment.labels_22%257D%255D%22))).

## Save your edits to the solution

Use any of the following methods to save your edits to the solution

---
**Download your solution in tar file**
* Click on the `File` menu
* Select `Download Workspace` to download the whole workspace on your local in compressed format.

---
**Save your solution to your git repository**

Set the remote url to your own repository
```bash 
git remote set-url origin [your-own-repo-url]
```

Review the modified files, commit and push to your remote repository branch.
<walkthrough-inline-feedback></walkthrough-inline-feedback>
