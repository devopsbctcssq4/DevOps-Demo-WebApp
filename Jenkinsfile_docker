pipeline {
     environment {
	     
    //login id/docker reposotory defined in Jenkins followe by repository name
    registry = "arunsaxena01/avncommunication"
    //credential = Id given jenkins
    registryCredential = 'dockerhub'
    dockerImage = ''
    //checked if any container is running with same name node-app container Id will store same & stage cleanup will close that container
    containerId = sh(script: 'docker ps -aqf "name=myApp"', returnStdout: true)
  }  
    
    agent any
    tools {
      //   maven 'Maven3.6.3'
       maven 'Maven'
    }

    stages {
     
     stage('Build') {
          steps {
                sh "mvn -B -DskipTests clean package"
            }
        }
        
            stage('Building image') {
     steps{
        script {
                      //will pisck registry from variable defined
          dockerImage = docker.build registry + ":$BUILD_NUMBER"
        }
      }
    }
                    stage('Push Image') {
     steps{
       script {
            docker.withRegistry( '', registryCredential ) {
            dockerImage.push()
          }
        }
      }
    }
         
      stage('Cleanup') {
      when {
                not { environment ignoreCase: true, name: 'containerId', value: '' }
        }
      steps {
        sh 'docker stop ${containerId}'
        sh 'docker rm ${containerId}'
      }
    }
                  
      stage('Run Container') {
      steps {
   //     sh 'docker run --name=myapp -d -p 8081:8000 $registry:$BUILD_NUMBER &'
          sh 'docker run -d -p 8081:8080 --name=myApp $registry:$BUILD_NUMBER'
      }
             } 
	    stage('Request approval') { // Raise change request
            steps {
                echo 'Raise change request...'
                jiraSendDeploymentInfo(site:'aksservicedesk.atlassian.net',
                        environmentId:'prod-1',
                        environmentName:'prod-1',
                        environmentType:'production',
                        state:"pending",
                        enableGating:true,
			issueKeys: ['JNG-11'],						
                        serviceIds: [
                          'b:YXJpOmNsb3VkOmdyYXBoOjpzZXJ2aWNlLzE0Yjc1NWYyLTdiYTEtMTFlYi05NjRjLTBhYmUzZjRhNjYwMS9jZmU3YTk0ZS04NjhjLTExZWItOGExNy0wYWJlM2Y0YTY2MDE='
                        ]
                    )
            }
        }
        stage("Approval gate") { // Check change request status
            steps {
                retry(20) { // Poll every 30s for 10min
                    waitUntil { 
                        sleep 30
                        checkGatingStatus(
                          site:'aksservicedesk.atlassian.net', 
                          environmentId:'prod-1'
                        )
                    }
                }   
            }
        }
	    
	    stage('Deploy App in Kuberneter cluster') {
             
            steps {
               withCredentials([usernamePassword(credentialsId: 'acr-credentials', usernameVariable: 'ACR_ID', passwordVariable: 'ACR_PASSWORD')]) {
		//sh 'kubectl apply -f deployment.yaml'	
		 sh 'kubectl set image -n default deployment/myapp myapp=arunsaxena01/avncommunication:$BUILD_NUMBER'  
		 echo 'kubectl set image -n default deployment/myapp myapp=manivannanmari/dockerdemocasestudy1:$BUILD_NUMBER'
		}
 
            }
        }     
	
	    stage("Production Deployment notification to Jira") {
            steps {
                echo "Deploying to production!!"
            }
            post {
              always {
                sh 'sleep 2'
              }
              // Notify Jira based on deployment step result
              success {
                jiraSendDeploymentInfo (
                        site: 'aksservicedesk.atlassian.net',
                        environmentId: 'prod-1',
                        environmentName: 'prod-1',
                        environmentType: 'production',
                        state: 'successful',
			issueKeys: ['JNG-11'],
                        serviceIds: [
                          'b:YXJpOmNsb3VkOmdyYXBoOjpzZXJ2aWNlLzE0Yjc1NWYyLTdiYTEtMTFlYi05NjRjLTBhYmUzZjRhNjYwMS9jZmU3YTk0ZS04NjhjLTExZWItOGExNy0wYWJlM2Y0YTY2MDE='
                        ]
                      )
              }
            }
		 }	
    }
}
