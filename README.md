# Jenkinsfile

This project is to highlight the build pipeline which will build and deploy a project using Jenkins, Docker, and Terraform. Most of the "myapp" files are part of the generic base angular project built in [example-anguluar-project-docker](https://github.com/DimsumPanda/example-angular-project-docker).

There are two types of pipeline syntax in Jenkins: scripted and declarative. Both lends itself to much of the functionality in the Apache Groovy language. Pipelines, for the most part, are executed synchronously, but there is flexibiliy to extend and leverage parallel stages (in scripted syntax).

The Jenkinsfile in this project is written in scripted pipeline for its flexibiility and extensibilty. 

## Objective
Create an autoscaling group (ASG) of EC2 instances. The pipeline will build the application and push the image into Dockerhub. The image will then be used to spin up a container in each EC2 instance in the ASG. The loadbalancer would expose the application through its DNS.

## Prerequisites
- Jenkins instance
- Jenkins have terraform version 0.12.x installed
- Github account
- Dockerhub account
- AWS account

## 