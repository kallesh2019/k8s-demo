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
        EMAIL_TO = 'kalleshdevops@gmail.com'		
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
           dockerImage = docker.build "${IMAGE_REPO_NAME}:${IMAGE_TAG}","./docker"
        }
      }
    }
    	stage('Pushing to ECR') {
           steps{  
            script {
                sh "docker tag ${IMAGE_REPO_NAME}:${IMAGE_TAG} ${REPOSITORY_URI}:$IMAGE_TAG"
                sh "docker push ${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_DEFAULT_REGION}.amazonaws.com/${IMAGE_REPO_NAME}:${IMAGE_TAG}"
         }
        }
      }
	    stage('Deploy App on k8s') {
           steps {
            sshagent(['ssh-k8s-master']) {
            sh "scp -o StrictHostKeyChecking=no java-app-deployment.yaml ec2-user@13.232.166.238:/home/ec2-user"
            script {
                try{
                  sh "ssh ec2-user@13.232.166.238 kubectl apply -f ."
                }catch(error){
                    sh "ssh ec2-user@13.232.166.238 kubectl create -f ."
            }
}
        }
    }
}  
    }
}
post
{ 
  always
  { 
    cleanWs()
	sh "echo This will always run"
	}
	sucess
	{ 
	script
	{
	 sh "echo This will always run"
	 email subject: "Build ${BUILD_NUMBER} status report for '${env.JOB_NAME}'", subject: 'Build Success in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER',to: "${EMAIL_TO}"
	 }
  }
  	failure
	{ 
	script
	{
	 sh "echo This will always run"
	 email subject: "Build ${BUILD_NUMBER} status report for '${env.JOB_NAME}'", subject: 'Build failure in Jenkins: $PROJECT_NAME - #$BUILD_NUMBER',to: "${EMAIL_TO}"
	 }
  }
 }
