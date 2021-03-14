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
     
    stage('Static Code Analysis') {
	    steps {
     withSonarQubeEnv(credentialsId: 'sonar', installationName: 'sonarqube') {         
	sh '/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/Maven/bin/mvn -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=sonar -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.sources=. sonar:sonar -Dsonar.host.url=http://23.96.91.93:9000'	
	}
    }
    }
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
	    
	    stage('Deploy App in Kuberneter cluster') {
             
            steps {
               withCredentials([usernamePassword(credentialsId: 'acr-credentials', usernameVariable: 'ACR_ID', passwordVariable: 'ACR_PASSWORD')]) {
		//sh 'kubectl apply -f deployment.yaml'	
		 sh 'kubectl set image -n default deployment/myapp myapp=arunsaxena01/avncommunication:$BUILD_NUMBER'  
		 echo 'kubectl set image -n default deployment/myapp myapp=manivannanmari/dockerdemocasestudy1:$BUILD_NUMBER'
		}
 
            }
        }     

    }
}
