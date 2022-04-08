pipeline {
  agent any
    tools {
      maven 'maven-jenkins'
      jdk 'JDK-11-jenkins'
    }
    environment{
        AWS_ACCOUNT_ID="466243422136"
        AWS_DEFAULT_REGION="ap-south-1" 
        IMAGE_REPO_NAME="devops-pipeline"
        IMAGE_TAG="latest"
        REPOSITORY_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}"	    
    }
    stages{
        stage('Logging into AWS ECR') {
            steps {
                script {
                sh "aws ecr get-login-password --region ${AWS_DEFAULT_REGION} | docker login --username AWS --password-stdin ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com"
                }
             }
        }	         
        stage('Build maven') {
            steps { 
                    sh 'pwd'      
                    sh 'mvn  clean install package'
	          }   
            }
        
        stage('Copy Artifact') {
           steps { 
                   sh 'pwd'
		   sh 'cp -r target/*.jar docker'
           }
        }
        stage('Building image') {
           steps{
           script {
           dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}"
        }
      }
    }
    }
}
