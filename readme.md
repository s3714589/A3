# ACME App

Follow step by step to create a Continuous Deployment of ACME app pipeline

# Deployment step 

### Provide aws credentials to circleci 
1. Login to circleci
2. Go to Project Settings
3. Go to Environment Variables
4. Put credentials like following
<img src="/img/creds.png"></image>

### Create a HELM chart to deploy the application to Kubernetes  
I created HELM chart with templates of deployment and service inside /helm/sdo-app folder with values.yaml file store image and dbhost to be passed in.

### Deploy the application into a non-production environment
1. Deploy backend to aws from local machine
* Go to /environment directory
* Run script `Make up`. When deploying finished, you can get output similar to following:  
<img src="/img/back-end.png"></image>
* Run script `Make kube-up` to deploy kube. When deploying finished, you can get output similar to following:  
<img src="/img/create-kube.png"></image>
* Create 2 namespaces to kubenet by running following commands:
```
$> kubectl create namespace test
$> kubectl create namespace prod
```
2. Setting up all necessary variables
* Edit file /infra/terraform.tfvars by getting deployed VPC ID and Subnet ID, from aws and username, password, name variables need to be matched with the value provided to env in /helm/sdo-app/deployment.yaml as following:
```
subnet_ids = [
  "subnet-05d067ef3cf000e43",
  "subnet-0fc088c4b331a097f"
]
username = "postgres"
password = "password"
vpc_id = "vpc-00fb8b13483bac3fa"
name = "servian"
```
* Copy from backend terraform output of dynamoDb_lock_table_name,state_bucket_name and modify /infra/Makefile as following
```
terraform init --backend-config="key=state/${ENV}.tfstate" --backend-config="dynamodb_table=RMIT-locktable-5dgbn0" --backend-config="bucket=rmit-tfstate-5dgbn0"
```
* Copy from backend terraform output of kops_state_bucket_name and modify /.circleci/config.yaml Configure environment as following
```
environment:
    KOPS_STATE: rmit-kops-state-5dgbn0
```
* Copy from backend terraform output of ecr_url and modify /.circleci/config.yaml package job as following
```
environment:
    ECR: 104363569144.dkr.ecr.us-east-1.amazonaws.com
    reponame: app
    NODE_ENV: production
```
* Successful pipeline:
<img src="/img/test.png"></image>
### Change the end-to-end test to run against the non-production environment 
* Successful pipeline:
<img src="/img/e2e.png"></image>
### Deploy the application into a production environment 
* Whole pipeline with approval job to continue with deploying to production environment
<img src="/img/pipeline.png"></image>
* Successful pipeline:
<img src="/img/prod.png"></image>
### Integrate logging to the solution so any logs from the Kubernetes cluster is automatically stored in AWS CloudWatch for the future
<img src="/img/cloud.png"></image>  
The code I use to deploy the logging solution to the cluster in file fluentd.yaml

