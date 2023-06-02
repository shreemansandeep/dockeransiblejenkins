pipeline{
    agent any
    
    /*
    tools {
      maven 'maven3'
    }
    */
    
    environment {
      DOCKER_TAG = getVersion()
    }
    
    stages{
        
        stage('SCM'){
            steps{
                git credentialsId: 'GitCred', 
                    url: 'https://github.com/shreemansandeep/dockeransiblejenkins.git'
            }
        }
        
        stage('Maven Build'){
            steps{
                sh "mvn clean package"
            }
        }
        
        stage('Docker Build'){
            steps{
                sh "docker build . -t dockersandheep/$JOB_NAME:${DOCKER_TAG} "
            }
        }
        
        stage('DockerHub Push'){
            steps{
                withCredentials([string(credentialsId: 'DockerHubCred', variable: 'dockerHubPwd')]) {
                    sh "docker login -u dockersandheep -p ${dockerHubPwd}"
                }
                
                sh "docker push dockersandheep/$JOB_NAME:${DOCKER_TAG}"
            }
        }
       
        
        stage('Docker Deploy'){
            steps{
              ansiblePlaybook credentialsId: 'dev-server', disableHostKeyChecking: true, extras: "-e DOCKER_TAG=${DOCKER_TAG}", installation: 'ansible', inventory: 'dev.inv', playbook: 'deploy-docker.yml'
            }
        } 
        
    }
}

def getVersion(){
    def commitHash = sh label: '', returnStdout: true, script: 'git rev-parse --short HEAD'
    return commitHash
}
