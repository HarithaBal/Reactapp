pipeline {
    agent any
        environment {
        registry = "519852036875.dkr.ecr.us-east-2.amazonaws.com/reactapp"
    }
     stages{
         stage('Build') {
            steps {
                nodejs(nodeJSInstallationName: 'Nodejs'){
                    sh 'npm install'
                }
           }
        }
         stage("SonarQube analysis") {
            agent any
            steps {
              withSonarQubeEnv('Jenkins-Sonar-Integration') {
                sh 'npm install sonar-scanner'
		 sh'npm i sonar-scanner --save-dev'
		  sh 'npm run sonar-scanner'    	
              }
            }
          }
	     stage("Quality Gate") {
            steps {
              timeout(time: 1, unit: 'HOURS') {
                waitForQualityGate abortPipeline: true
                  }
                 }
	     }
	    
	     
        stage('Building image in EC2') {
      steps{
        script {
            docker.withRegistry( 'https://registry.hub.docker.com/','Jenkins-Docker-Integration'){
             myImage = docker.build ("reactapp:latest")
            }
        }
      }
    }
    stage('Build Registry') {
      steps{
        script {
          dockerImage = docker.build registry
        }
      }
    }
  
     // Uploading Docker images into AWS ECR
    stage('Pushing to ECR') {
     steps{  
         script {
                sh 'aws ecr get-login-password --region us-east-2 | docker login --username AWS --password-stdin 519852036875.dkr.ecr.us-east-2.amazonaws.com'
                sh 'docker push 519852036875.dkr.ecr.us-east-2.amazonaws.com/reactapp:latest'
         }
        }

      }
	     stage('Deploy to K8S'){
		steps{
		 sshagent(['Kubernetes-Jenkins-Integration']) {
   		  sh 'scp -o StrictHostKeyChecking=no reactapp.yml ubuntu@3.141.5.128:/home/ubuntu'
		 script{
		  try{
			sh 'ssh ubuntu@3.141.5.128 kubectl apply -f .'
			}catch(error){
			sh 'ssh ubuntu@3.141.5.128 kubectl create -f .'
				    }
			      }
		           }
			}
		 
		 }
	   
      
    }
     
   }
