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
        nexusUrl = 'http://localhost:8081'
    }
    parameters {
        booleanParam(name: 'deploy', defaultValue: false, description: 'deploy the application to the environment')
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
                zip -q -r backend.${appVersion}.zip * -x Jenkinsfile -x backend.${appVersion}.zip
                ls -ltr              
               """
            }
        }
        stage('sonar scan') {
            environment {
                scannerHome = tool 'sonar-6.0' //scannar cli
            }
            steps{
                script {
                    withSonarQubeEnv('sonar-6.0') {  
                        sh "${scannerHome}/bin/sonar-scanner 
                        }
                    }
            }
        }
        stage('Quality Gate') {
            steps {
                timeout(time: 30, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Upload to nexus') {
            steps {
               sh """
                curl --upload-file backend.${appVersion}.zip http://localhost:8081/repository/expense-backend/backend.${appVersion}.zip
               """
            }
        }
        stage('Nexus artifact uploader') {
            steps {
               script {
                   nexusArtifactUploader(
                        NexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: '${nexusUrl}',
                        groupId: 'com.expense',
                        version: "${appVersion}",
                        repository: 'backend',
                        credentialsId: 'nexus-auth',
                        artifacts: [
                            [artifactId: 'backend', classifier: '', file: "backend-${appVersion}.zip", type: 'zip']
                        ]
                    )
               }
            }
        }
        stage('Deploy') {
        when {
            expression { params.deploy }
            }
            steps {
                script {
                    def params = [
                            string(name: 'appVersion', value: "${appVersion}")
                        ]   
                        build job: 'deploy-backend', parameters: params, wait: false
                    }
            }
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
