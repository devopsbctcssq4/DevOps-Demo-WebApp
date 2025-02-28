node {
  
    // Getting Artifactory server instance
    
    	def server = Artifactory.server "Artifcatory1"
    
    // Creating an Artifactory Maven instance.
    
    	def rtMaven = Artifactory.newMavenBuild()
    	def buildInfo 
	
    // Getting Tools  from Jenkins configuration
    
	rtMaven.tool = "Maven"
	
    // Setting Artifactory repositories for dependencies resolution and artifacts deployment.
    
        rtMaven.deployer releaseRepo:'libs-release-local', snapshotRepo:'libs-snapshot-local', server: server
        rtMaven.resolver releaseRepo:'libs-release', snapshotRepo:'libs-snapshot', server: server

	// Sending Slack message for pipeline start 
	slackSend channel: 'alerts', message: "Pipeline Started ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", teamDomain: 'devopsbc', tokenCredentialId: 'slack'
   
	// Cloning Soure code from github	    
        stage('Clone source Code') {
        git url: 'https://github.com/devopsbctcssq4/DevOps-Demo-WebApp.git'
    }
    

//  Static code Analysis using Sonarqube  
    stage('Static Code Analysis') {
        withSonarQubeEnv(credentialsId: 'sonar', installationName: 'sonarqube') { // You can override the credential to be used
       	//sh 'mvn clean package sonar:sonar -Dsonar.host.url=http://52.152.224.93// -Dsonar.sources=. -Dsonar.tests=. -Dsonar.test.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.exclusions=**/test/java/servlet/createpage_junit.java'
        
	sh '/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/Maven/bin/mvn -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=sonar -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.sources=. sonar:sonar -Dsonar.host.url=http://23.96.91.93:9000'	
	}
    				}
   // Building web app using Maven build	
    stage('Build Web App') {
        buildInfo = rtMaven.run pom: 'pom.xml', goals: 'clean install', buildInfo: buildInfo
    }

	// Sending Build notificaiton to Jira
	jiraSendBuildInfo branch: 'JNG-2', site: 'aksservicedesk.atlassian.net'
    
	//Deploying web app to QA env
	stage('Deploy to Test') {
	deploy adapters: [tomcat8(credentialsId: 'tomcat-1', path: '', url: 'http://40.112.56.191:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
	
		//Send QA Deployment info to Jira
	jiraSendDeploymentInfo environmentId: 'test-1', environmentName: 'testing', environmentType: 'testing',  site: 'aksservicedesk.atlassian.net',issueKeys: ['JNG-2'], serviceIds: [''],state: 'successful'

    }

	// Storing Artifacts in Artifactory
    stage('Store  Artifacts') {
        server.publishBuildInfo buildInfo
    }
	
	// Perfrom UI Test
      stage('Functional (UI) Test') {
        buildInfo = rtMaven.run pom: 'functionaltest/pom.xml', goals: 'test'
	publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test Report', reportTitles: ''])
    }
     
	// DO Performance Test of app using Blazemeter
       stage('Performance Test') {
    	echo 'Running BlazeMeterTest' 
    //blazeMeterTest credentialsId: 'Blazemeter', testId: '9014498.taurus', workspaceId: '756635'
    }
	 
	//Package,Build Docker Image and Push
	
	stage('Package,Build Docker Image and Push') {

                sh "/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/Maven/bin/mvn package"
		sh 'docker build -t arunsaxena01/avncommunication:$BUILD_NUMBER /var/lib/jenkins/workspace/DevOps-Demo-WebApp' 
   
        withDockerRegistry([ credentialsId: "dockerhub", url: "" ]) {
          sh  'docker push arunsaxena01/avncommunication:$BUILD_NUMBER' 
        }

        }
	
           stage('Prod deployment Approval Gate') { // Raise change request
            
                echo 'Raise change request...'
                jiraSendDeploymentInfo(site:'aksservicedesk.atlassian.net',
                        environmentId:'prod-1',
                        environmentName:'prod-1',
                        environmentType:'production',
                        state:"pending",
                        enableGating:true,
			issueKeys: ['JNG-3'],						
                        serviceIds: [
                          'b:YXJpOmNsb3VkOmdyYXBoOjpzZXJ2aWNlLzE0Yjc1NWYyLTdiYTEtMTFlYi05NjRjLTBhYmUzZjRhNjYwMS9jZmU3YTk0ZS04NjhjLTExZWItOGExNy0wYWJlM2Y0YTY2MDE='
                        ]
                    )
            
         // Check change request status
            
                retry(20) { // Poll every 30s for 10min
                    waitUntil { 
                        sleep 10
                        checkGatingStatus(
                          site:'aksservicedesk.atlassian.net', 
                          environmentId:'prod-1'
                        )
                    
                }   
            }
        }
	
	stage('Deploy Web App in Production K8s cluster') {
               
		// sh 'kubectl apply -f deployment.yaml'	
		 sh 'kubectl set image -n default deployment/myapp myapp=arunsaxena01/avncommunication:$BUILD_NUMBER'  
		// Sending Build notificaiton to Jira
		jiraSendBuildInfo branch: 'JNG-3', site: 'aksservicedesk.atlassian.net'
               sh 'sleep 2'
              
              // Notify Jira based on deployment step result
              
                jiraSendDeploymentInfo (
                        site: 'aksservicedesk.atlassian.net',
                        environmentId: 'prod-1',
                        environmentName: 'prod-1',
                        environmentType: 'production',
                        state: 'successful',
			issueKeys: ['JNG-3'],
                        serviceIds: [
                          'b:YXJpOmNsb3VkOmdyYXBoOjpzZXJ2aWNlLzE0Yjc1NWYyLTdiYTEtMTFlYi05NjRjLTBhYmUzZjRhNjYwMS9jZmU3YTk0ZS04NjhjLTExZWItOGExNy0wYWJlM2Y0YTY2MDE='
                        ]
                      )  
		 }
		

	
	// Perform Sanity Test on Prod
        stage('Sanity Test in Production') {
		echo 'Sanity Test'
		sh 'sleep 10'
        buildInfo = rtMaven.run pom: 'Acceptancetest/pom.xml', goals: 'test'
	publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
    			  }
	//jiraSendDeploymentInfo enableGating: true, environmentId: '', environmentName: '', environmentType: 'production', issueKeys: ['TDB-1'], serviceIds: [''], site: 'fresco3.atlassian.net', state: 'successful'
	//Send Slack message for Pipeline Completion
	slackSend channel: 'alerts', message: "Pipeline Completed ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", teamDomain: 'devopsbc', tokenCredentialId: 'slack'
	//cleanWs()
}
