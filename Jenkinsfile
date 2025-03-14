pipeline {
    agent any
    stages {
        stage('Checkout git') {
            steps {
               checkout scmGit(branches: [[name: params.REPO_BRANCH]], extensions: [], userRemoteConfigs: [[credentialsId: 'hari', url: params.REPO_URL]])
            }
        }
        stage('Build') {
            tools {
                jdk 'java-17'
            }
            steps {
                sh 'mvn clean package'
            }
        } 
        stage('SAST-Analysis') {
            tools {
                jdk 'java-17'
            }
            environment {
                scannerHome = tool 'SonarQubeScanner'
                projectName = 'DevSecOps-project'
            }
            steps { 
                withSonarQubeEnv('sonar_1') {
                    sh """
                        export JAVA_HOME=\$JAVA_HOME
                        export PATH=\$JAVA_HOME/bin:\$PATH
                        ${scannerHome}/bin/sonar-scanner \
                        -Dsonar.projectKey=${projectName} \
                        -Dsonar.sources=src \
                        -Dsonar.java.binaries=.
                    """
                }
            }
        }   
        stage('pulish') {
            tools {
                jdk 'java-17'
            }
            steps {
                nexusArtifactUploader artifacts: [[artifactId: 'DevSecOps-project', classifier: '', file: '/var/lib/jenkins/workspace/Intenship/Hari/DevSecOps-project/target/demo-0.0.1-SNAPSHOT.jar', type: 'jar']], credentialsId: 'nexus31', groupId: 'com.logicfocus', nexusUrl: '192.168.1.163:32007', nexusVersion: 'nexus3', protocol: 'http', repository: 'DevSecOps-project', version: '1.0.1'
            }
        }
        stage('upload artifact'){
            steps{
                archiveArtifacts artifacts: 'target/demo-0.0.1-SNAPSHOT.jar', onlyIfSuccessful: true , allowEmptyArchive: true
            }
        }
        stage('Artifact-Details') {
            steps {
                script {
                    def currentpath=pwd()
                    def path=currentpath.split("/")
                    def folderName = path[-3]
                    def buildData = [
                        buildNumber: env.BUILD_NUMBER,
                        jobName: env.JOB_NAME,
                        jobUrl: env.BUILD_URL,
                        gitCommit: sh(script: 'git rev-parse HEAD', returnStdout: true).trim(),
                        repository: sh(script: 'basename `git config --get remote.origin.url` .git', returnStdout: true).trim(),
                        "jenkins_folderName": "${folderName}" 
                    ]
                    writeFile file: "Artifact-details.json", text: groovy.json.JsonOutput.prettyPrint(groovy.json.JsonOutput.toJson(buildData))
                }
            }
        }
        stage('Read-Build-Json'){
            steps{
                archiveArtifacts artifacts: 'Artifact-details.json', onlyIfSuccessful: true , allowEmptyArchive: true
            }
        }
    }
}
      

    
