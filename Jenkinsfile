pipeline {
     environment {
    //login id/docker reposotory defined in Jenkins followe by repository name
    registry = "jimish22/avncommunication"
    //credential = Id given jenkins
    registryCredential = 'arunsaxena01'
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
        sh 'docker run -d -p 8081:8080 --name=myApp $registry:$BUILD_NUMBER &'
      }
             }   

    }
}
