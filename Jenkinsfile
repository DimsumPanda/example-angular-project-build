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
        stage("Testing"){}
    }
}
def pipelineFeatureBranch(){
    node("master"){
        cleanWs()
        stage("Testing"){}
    }
}
def pipelinePullRequest(){
    node("master"){
        cleanWs()
        stage("Testing"){}
    }
}
