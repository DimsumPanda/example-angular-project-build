// Jenkins Credential IDs used in the file
GITHUB_ACCOUNT_CREDSID = "github-account"

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
        tag = env.BRANCH_NAME.minus("feature/")
        dockerfile_path = "myapp/Dockerfile"
        build_path = "myapp"
        stageDockerBuild("Docker Build"){
            sh "docker build --no-cache --tag ${tag} -f ${dockerfile_path} ${build_path}"
        }
        // cleanWs()
    }
}
def pipelineFeatureBranch(){
    node("master"){
        cleanWs()
        setUpJobProperties()
        stage("Testing"){}
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

// ================================================
// Functions
// ================================================
def setUpJobProperties(){
    properties([buildDiscarder(logRotator(artifactDaysToKeepStr: '', artifactNumToKeepStr: '', daysToKeepStr: '2', numToKeepStr: '5'))])
}