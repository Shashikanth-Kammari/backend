pipeline {
    agent {
        label 'AGENT-1'
    }
    options {
        timeout(time: 30, unit: 'MINUTES')
        disableConcurrentBuilds()
        ansiColor('xterm')
    }
    evironment {
        appVersion = ''
    }
    
    stages {
        stage('read the version') {
            steps {
               script {
                   def version = readJSON file: 'package.json'
                   appVersion = version.version
                   echo "Version is ${appVersion}"
               }
            }
        }
        stage('Install Dependencies') {
            steps {
               sh """
               npm install
               ls -ltr
               echo "Version is ${appVersion}"
               """
            }
        }

        stage('build') {
            steps {
               sh """
                zip -r backend.${appVersion}.zip * -x Jenkinsfile -x backend.${appVersion}.zip
                ls -ltr              
               """
            }
        }
        
    post { 
        always { 
            echo 'I will always say Hello again!'
            deleteDir()  #it will delete the workspace after the build run
        }
        success { 
            echo 'I will run when pipeline is success'
        }
        failure { 
            echo 'I will run when pipeline is failure'
        }
    }
}