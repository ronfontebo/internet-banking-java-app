
// jenkins declarative pipeline script for beers-of-the-world-java-webapp
//=======================================================================


pipeline {
  agent any
  tools {
    maven "maven"
  }
  triggers {
    githubPush()
  }
  environment {
    PROJECT_ID = 'internet-banking-java-app'
  }
  stages {
    stage('1. code checkout') {    
      steps {
        sh "echo cloning the lastest code version"
        git branch: 'main', 
        credentialsId: 'git-token', 
        url: 'https://github.com/ronfontebo/${PROJECT_ID}'
        sh "echo clonning successful"
      }
    }
  
    stage('2. maven build') {
      steps {
        sh "echo validation, compilation, testing and packaging"
        sh "echo testing successful and ready to package"
        sh "mvn clean package"        
      }
    }

    stage('3. sonarqube code quality analysis') {
      environment {
        SONAR_CREDS = credentials('sonarqube-token')
        SONAR_IP    = credentials('sonarqube-ip')
      }  
      steps {
        sh "echo performing code quality analysis"
        sh 'mvn sonar:sonar \
          -Dsonar.projectKey=${PROJECT_ID} \
          -Dsonar.host.url=${SONAR_IP} \
          -Dsonar.login=${SONAR_CREDS}'
        sh "echo code quality successful and ready to upload"
      }
    } 
  
   /* post {
      always {
        sh "echo notifying slack channel on build status"
        slackSend botUser: true, channel: '#beers-of-the-world-java-webapp', message: 'Build completed and successful. Ready to upload backup to nexus and deploy to tomcat servers respectively. Only 4/23 build attempts have been successful so far. : ${env.JOB_NAME} ${env.BUILD_NUMBER}  ', tokenCredentialId: 'slack-token'     
      }
    } */ 

    stage('4. upload to nexus') {   
      steps {
        sh "echo uploading artifacts"
        sh "mvn deploy"
      }
    }

    stage('5. deploy to tomcat') {      
      environment {
        TOMCAT_IP = credentials('tomcat-ip')
      }  
      steps {      
        sh "echo deploying to production server"
        sshagent(['agentcredentials']) {
        sh 'scp -i apache-key-pair.pem target/*.war ec2-user@${TOMCAT_IP}:/opt/tomcat9/webapps/${PROJECT_ID}.war'
        }
      }
    } 
  }
  
  post {
    always {
      sh "echo notifying slack channel on deployment status"
      slackSend botUser: true, channel: '#${PROJECT_ID}', message: 'Deployment completed and successful : ${env.JOB_NAME} ${env.BUILD_NUMBER}  ', tokenCredentialId: 'slack-token'
    }
  }
}  
 
  
// End of pipeline
//=================================================================================================================================
