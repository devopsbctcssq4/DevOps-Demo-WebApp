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
    stage('Clone source') {
        git url: 'https://github.com/arunsaxena01/DevOps-Demo-WebApp.git'
    }
    

//  Static code Analysis using Sonarqube  
    stage('Static Code Analysis') {
        withSonarQubeEnv(credentialsId: 'akssonartoken', installationName: 'sonarqube') { // You can override the credential to be used
       	//sh 'mvn clean package sonar:sonar -Dsonar.host.url=http://52.152.224.93// -Dsonar.sources=. -Dsonar.tests=. -Dsonar.test.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.exclusions=**/test/java/servlet/createpage_junit.java'
        
	sh '/var/lib/jenkins/tools/hudson.tasks.Maven_MavenInstallation/Maven/bin/mvn -Dsonar.test.exclusions=**/test/java/servlet/createpage_junit.java -Dsonar.login=admin -Dsonar.password=sonar -Dsonar.tests=. -Dsonar.inclusions=**/test/java/servlet/createpage_junit.java -Dsonar.sources=. sonar:sonar -Dsonar.host.url=http://52.152.224.93:9000'	
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
	deploy adapters: [tomcat8(credentialsId: 'tomcat-1', path: '', url: 'http://52.255.157.89:8080/')], contextPath: '/QAWebapp', war: '**/*.war'
	
		//Send QA Deployment info to Jira
	jiraSendDeploymentInfo environmentId: 'JNG-2', environmentName: 'testing', environmentType: 'testing',  site: 'aksservicedesk.atlassian.net',issueKeys: ['JNG-2'], serviceIds: [''],state: 'successful'

    }

	// Storing Artifacts in Artifactory
    stage('Store  Artifacts') {
        server.publishBuildInfo buildInfo
    }
	
	// Perfrom UI Test
      stage('UI Test') {
        buildInfo = rtMaven.run pom: 'functionaltest/pom.xml', goals: 'test'
	publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\functionaltest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'UI Test Report', reportTitles: ''])
    }
     
	// DO Performance Test of app using Blazemeter
       stage('Performance Test') {
    	echo 'Running BlazeMeterTest' 
    //blazeMeterTest credentialsId: '917117ed-d257-41d2-bb35-47dea13f959c', testId: '9014498.taurus', workspaceId: '756635'
    }
	
	//Deploy web App to Prod
        stage('Deploy to Prod') {
	      deploy adapters: [tomcat8(credentialsId: 'tomcat-1', path: '', url: 'http://13.68.144.119:8080/')], contextPath: '/ProdWebapp', onFailure: false, war: '**/*.war'
	     //jiraSendDeploymentInfo environmentId: 'Staging', environmentName: 'Staging', environmentType: 'staging', serviceIds: ['http://13.68.144.119:8080/ProdWebapp'], site: 'devopsbc.atlassian.net', state: 'successful'
	     
	      //Send Prod Deployment info to Jira
	      jiraSendDeploymentInfo environmentId: 'JNG-3', environmentName: 'production', environmentType: 'production',  site: 'aksservicedesk.atlassian.net',issueKeys: ['JNG-3'], serviceIds: [''],state: 'successful'
         }
	
	// Perform Sanity Test on Prod
        stage('Sanity Test') {
        buildInfo = rtMaven.run pom: 'Acceptancetest/pom.xml', goals: 'test'
	publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '\\Acceptancetest\\target\\surefire-reports', reportFiles: 'index.html', reportName: 'Sanity Test Report', reportTitles: ''])
    			  }
	//jiraSendDeploymentInfo enableGating: true, environmentId: '', environmentName: '', environmentType: 'production', issueKeys: ['TDB-1'], serviceIds: [''], site: 'fresco3.atlassian.net', state: 'successful'
	//Send Slack message for Pipeline Completion
	slackSend channel: 'alerts', message: "Pipeline Completed ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)", teamDomain: 'devopsbc', tokenCredentialId: 'slack'
}
