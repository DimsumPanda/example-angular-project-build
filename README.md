# Jenkinsfile

This project is to highlight the build pipeline which will build and deploy a project using Jenkins, Docker, and Terraform. Most of the "myapp" files are part of the generic base angular project built in [example-anguluar-project-docker](https://github.com/DimsumPanda/example-angular-project-docker).

There are two types of pipeline syntax in Jenkins: scripted and declarative. Both lends itself to much of the functionality in the Apache Groovy language. Pipelines, for the most part, are executed synchronously, but there is flexibiliy to extend and leverage parallel stages (in scripted syntax).

The Jenkinsfile in this project is written in scripted pipeline for its flexibiility and extensibilty. 

The intended users of the example are for people new to writing Jenkins Pipelines.