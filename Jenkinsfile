// Jenkins Credential IDs used in the pipeline
GITHUB_CREDSID = "github-account"
DOCKERHUB_CREDSID = "dockerhub-account"

// ================================================
// Branching Strategy
// ================================================
echo 'BRANCH_NAME: ' + env.BRANCH_NAME

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
        checkout scm
        
        dockerfile_path = "myapp/Dockerfile"
        build_path = "myapp/."
        registry = "thedimsumpanda/prod"
        image_name = "example-angular-project"
        tag = sh(returnStdout: true, script: "git describe --exact-match ${scmVars.GIT_COMMIT} || true").trim() // checks if there a git tag on the last commit

        stageImageBuild(image_name, tag, dockerfile_path, build_path)
        if(tag){
            stageImagePush(DOCKERHUB_CREDSID, registry, image_name, tag)
        }
        
        // cleanWs()
    }
}
def pipelineFeatureBranch(){
    node("master"){
        cleanWs()
        setUpJobProperties()
        checkout scm
        
        dockerfile_path = "myapp/Dockerfile"
        build_path = "myapp/."
        registry = "thedimsumpanda/dev"
        image_name = "example-angular-project"
        tag = env.BRANCH_NAME.minus("feature/")

        stageImageBuild(image_name, tag, dockerfile_path, build_path)
        stageImagePush(DOCKERHUB_CREDSID, registry, image_name, tag)
        
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
def stageImageBuild(image_name, tag, String dockerfile_path="Dockerfile", String build_path="."){
    stage("Image Build"){
        sh "docker build --no-cache --tag ${image_name}:${tag} -f ${dockerfile_path} ${build_path}"
    }
}
def stageImageScan(registry_url, image_name, tag){
    stage("Image Scan"){
        // some image scan here
        echo "Image Scan Completed"
    }
}
def stageImagePush(registry_id, registry, image_name, tag){
    stage("Image Push"){
        withCredentials([string(credentialsId: registry_id, variable: 'PASSWORD')]) {
            sh "docker tag ${image_name}:${tag} ${registry}/${image_name}:${tag}"
            sh 'docker login --username thdimsumpanda --password $PASSWORD'
            sh "docker push ${registry}/${image_name}:${tag}"
        }
    }
}
// ================================================
// Functions
// ================================================
def setUpJobProperties(){
    properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '2', numToKeepStr: '5'))])
}