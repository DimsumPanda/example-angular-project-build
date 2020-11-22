# Jenkinsfile

This project focuses on the build pipeline which will build and deploy a project using Jenkins, Docker, and Terraform. Most of the "myapp" files are part of the generic base angular project built in [example-anguluar-project-docker](https://github.com/DimsumPanda/example-angular-project-docker).

There are two types of pipeline syntax in Jenkins: scripted and declarative. Both lends itself to much of the functionality in the Apache Groovy language. Pipelines, for the most part, are executed synchronously, but there is flexibiliy to extend and leverage parallel stages (in scripted syntax).

The Jenkinsfile in this project is written in scripted pipeline for its flexibiility and extensibilty. 

## Objective
Create an autoscaling group (ASG) of EC2 instances. The pipeline will build the application and push the image into Dockerhub. The image will then be used to spin up a container in each EC2 instance in the ASG. The loadbalancer would expose the application through its DNS.

Feature branches are intended to be pushed as a separate image from the Master Branch; however, this pipeline can be modified and customized to meet your needs.

## Prerequisites
- Jenkins instance (make sure it has enough compute/memory to run the file, if you run into issues executing the pipeline, try increasing the Jenkins machine)
- Jenkins have terraform version 0.12.x installed
- Github account
- Dockerhub account
- AWS account

## Get Started - AwS and Jenkins Credentials
1. Fork or clone this repo so you can modify and save in your own Github repository.
2. Create an AWS Access Key if you haven't done so already. If you need help setting up, you can reference the AWS documentation [here](https://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html).
3. Save access key credentials into Jenkins Credentials:
- Save AWSAccessKeyId as a secret-text credential with credentials ID: `AWS_ACCESS_KEY_ID`
- Save AWSSecretKey as a secret-text credential with credentials ID: `AWS_SECRET_ACCESS_KEY`
These two pieces of data will be used for terraform deployment to access AWS. Make sure the IAM user has access to create resources in the account.
4. Next save the Dockerhub credentials id as a username-password Jenkins Credential with credentials ID:`dockerhub-account`
5. save Github credentials as username-password Jenkins Credential with credentails ID: "github-account" 
6. For this example, I would like to be able to SSH into the machines, so I have saved a key pair called `jenkins-sshkey` and added the sshkey locally. This key is added to the terraform for the EC2 instances, if you would like replace it with another name, one way you can override the "jenkins-sshkey" name is by adding this at the end of line 127 and line 137 `-var='sshkey_name=<your-key-name>` 

### Modify Jenkinsfile
**Master Pipeline**
We'll modify the Jenkinfile in this repo for the master pipeline. We are using Github flow which consists of one master branch and feature branches that will be pulled in when they are ready.
The master pipeline will build an image each time it's run, but it will not push the image to the registry and deploy unless it's been git tagged.

**Feature Pipeline**
Feature pipeline will allow multiple feature branches to each deploy into its own cloud environment. The branch name after the "feature/" naming convention will be used for naming resources and tagging resources.

1. Create a feature branch, e.g. `git checkout -b "feature/test"`
2. Modify the TERRAFORM_PATH if needed. If you had set it on the $PATH, edit the path and replace line 3 with "terraform"
2. Modify the pipelineMasterBranch():
- replace line 32 with the name of your DockerHub registry which would be your Dockerhub account name.
3. Modify the pipelineFeatureBranch():
- replace line 64 with the name of your Dockerhub registry which would be your Dockerhub account name.
4. You will need to create the registries on Dockerhub, feel free to modify the "image_name" to match the repository name you create in your Dockerhub account.
5. Save your changes and commit the code to your own Github repository

We will be using a public Github repo's terraform files to deploy the ASG, feel free to check it out [here](https://github.com/DimsumPanda/example-angular-project-deploy.git). If you would like to modify the terraform files used here (sorry it's not a module right now), you can clone/fork the repo into your own repository and then replace the URL in the Jenkinsfile with your own.

### Set up the Jenkins Job
1. Click on "New Item"
- Give your job a valid name and create it as a Multibranch Pipeline
2. Configure job
- Select Git as the Branch Source:
    - Project Repository: the SCM of your repo where you have forked or cloned this repo
    - Credentials: github-account
    - Mode: by Jenkinsfile
        - Script Path: Jenkinsfile
3. Click Save

### Run the pipeline
1. If you click scan multibranch pipeline now, you should see the feature branch you created start to run after it discovers the Jenkinsfile.
2. Master branch may also run, you can stop it for now or let it run, but it will have errors since you haven't merged your changes yet.
3. The pipeline will build the image with the Dockerfile in the myapp folder, tag and push it to DockerHub, and then deploy resources to AWS.
4. At the end of the terraform apply, you should see the DNS of the loadbalancer with port 80 open. You can access the example application by copy and pasting the DNS URL into your browser. If you take a look inside your AWS account, you will also find the EC2s spun up with the ASG. You can go to any of the public ip addresses of the EC2s and see the app at <ip-address>:8080

After deploying the application, it will take a few minutes for the instance to start running and executing the cloud-init script, but it should come up within 1-3 minutes