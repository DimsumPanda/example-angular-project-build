// Jenkins Credential IDs used in the pipeline
DOCKERHUB_CREDSID = "dockerhub-account"
TERRAFORM_PATH    = "/usr/local/bin/terraform"
// ================================================
// Branching Strategy
// ================================================

if("${env.BRANCH_NAME}".matches("master")){
    echo 'Master Branch Pipeline'
    pipelineMasterBranch()
} else if("${env.BRANCH_NAME}".matches("feature/(.*)")){
    echo 'Feature Branch Pipeline'
    pipelineFeatureBranch()
} else if("${env.BRANCH_NAME}".matches("PR(.*")){
    echo 'Pull Request Pipeline'
    pipelinePullRequest()
} else {
    echo "No branch action enabled"
}
// ================================================
// Pipeline Definitions
// ================================================

def pipelineMasterBranch(){
    node("master"){
        cleanWs()
        setUpJobProperties()
        def scmVars = checkout scm
        
        dockerfile_path = "myapp/Dockerfile"
        build_path = "myapp/."
        registry = "thedimsumpanda"
        image_name = "example-angular-project"
        // checks if there a git tag on the last commit
        tag = sh(returnStdout: true, script: "git describe --exact-match ${scmVars.GIT_COMMIT} || true").trim() 
        tag = tag.replaceAll("\\.","-")
        tfstatefile_key = "global/s3/${image_name}-${tag}.tfstate"

        stageImageBuild(image_name, dockerfile_path, build_path)

        if(tag){
            stageImagePush(DOCKERHUB_CREDSID, registry, image_name, tag)
            dir('deploy'){
                checkoutSCM("https://github.com/DimsumPanda/example-angular-project-deploy.git")
                stageTerraformInit(tfstatefile_key)
                stageTerraformDestroy(tfstatefile_key,image_name,tag)
                stageTerraformApply(tfstatefile_key,image_name,registry)
            }
        } else {
            echo "No git tag attached to the commit, image is not pushed to repository. No resources will be deployed."
        }
        

    }
}
def pipelineFeatureBranch(){
    node("master"){
        cleanWs()
        setUpJobProperties()
        checkout scm
        
        dockerfile_path = "myapp/Dockerfile"
        build_path = "myapp/."
        registry = "thedimsumpanda"
        image_name = "example-angular-project-dev"
        tag = env.BRANCH_NAME.minus("feature/")
        tfstatefile_key = "global/s3/${image_name}-${tag}.tfstate"

        stageImageBuild(image_name, dockerfile_path, build_path)
        stageImagePush(DOCKERHUB_CREDSID, registry, image_name, tag)
        dir('deploy'){
            checkoutSCM("https://github.com/DimsumPanda/example-angular-project-deploy.git")
            stageTerraformInit(tfstatefile_key)
            stageTerraformDestroy(tfstatefile_key,image_name,tag)
            stageTerraformApply(tfstatefile_key,image_name,tag,registry)
        }
        
    }
}
def pipelinePullRequest(){
    node("master"){
        cleanWs()
        setUpJobProperties()
        stage("Testing"){}
    }
}
// ================================================
// Stages
// ================================================
def stageImageBuild(image_name, String dockerfile_path="Dockerfile", String build_path="."){
    stage("Image Build"){
        sh "docker build --no-cache --tag ${image_name} -f ${dockerfile_path} ${build_path}"
    }
}
def stageImagePush(registry_credsid, registry, image_name, tag){
    stage("Image Push"){
    withCredentials([usernamePassword(credentialsId: registry_credsid, passwordVariable: 'PASSWORD', usernameVariable: 'USERNAME')]) {
            sh "docker logout"
            sh "docker tag ${image_name} docker.io/${registry}/${image_name}:${tag}"
            sh 'docker login docker.io --username $USERNAME --password $PASSWORD'
            sh "docker push docker.io/${registry}/${image_name}:${tag}"
        }
    }
}

def stageTerraformInit(tfstatefile_key){
    stage("Terraform Init"){
        withCredentials([string(credentialsId: 'AWS_ACCESS_KEY_ID', variable: 'AWS_ACCESS_KEY_ID'), 
            string(credentialsId: 'AWS_SECRET_ACCESS_KEY', variable: 'AWS_SECRET_ACCESS_KEY')]) {
            
            // Set TF environment variables
            env.AWS_ACCESS_KEY_ID = AWS_ACCESS_KEY_ID
            env.AWS_SECRET_ACCESS_KEY = AWS_SECRET_ACCESS_KEY

            sh """
                ${TERRAFORM_PATH} init -backend-config='key=${tfstatefile_key}' 
            """
        }
    }
}
def stageTerraformDestroy(tfstatefile_key,image_name,tag){
    stage("Terraform Destroy"){
        sh """
            ${TERRAFORM_PATH} destroy --auto-approve -var='image_name=${image_name}' -var='image_tag=${tag}'
        """
    }
}
def stageTerraformApply(tfstatefile_key,image_name,tag,registry){
    stage("Terraform Apply"){
        sh """
            ${TERRAFORM_PATH} plan
            ${TERRAFORM_PATH} apply --auto-approve -var='image_name=${image_name}' -var='image_tag=${tag}' -var='image_registry=${registry}'
        """
    }
}
// ================================================
// Functions
// ================================================
def setUpJobProperties(){
    properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '2', numToKeepStr: '5'))])
}
def checkoutSCM( scm_url){
    checkout([$class: 'GitSCM', branches: [[name: 'main']],
    extensions: [[$class: 'CloneOption', shallow: true]], 
    userRemoteConfigs: [[url: scm_url]]])
}