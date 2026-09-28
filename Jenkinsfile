@Library('jenkins-shared-library')

// create variable of map type and set the values

def configMap = [
    type: "nodejsEKS"
    component: "backend"
    project: "expense"

]

if( ! env.BRANCH_NAME.equalsIgnoreCase('main')){
    pipeline-Decission.decidepipeline(configMap)
}
else{
    echo "proceed with CR or NON-PROD pipeline"
}

