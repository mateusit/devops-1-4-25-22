node {

  try {
    def mvnHome
    def pom
    def artifactVersion
    def tagVersion
    def retrieveArtifact

    stage('Prepare') {
      mvnHome = tool 'MAVEN'
    }

    stage('Checkout') {
       checkout scm
    }

    stage('Build') {
       if (isUnix()) {
          sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
       } else {
          bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
       }
    }

    stage('Unit Test') {
		junit '**/target/surefire-reports/TEST-*.xml'
	   
		archiveArtifacts 'target/*.war'
    }

    stage('Integration Test') {
      if (isUnix()) {
         sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean verify"
      } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean verify/)
      }
    }

     stage('Sonar') {
       if (isUnix()) {
		  sh "echo '**** STARTING SONARQUBE ANALYSIS ******'"	
		  sh "'${mvnHome}/bin/mvn' sonar:sonar -Dsonar.projectKey=com.example.devops:devops2 -Dsonar.host.url=http://mep-hygieia-docker-2.eastus2.cloudapp.azure.com:9000  -Dsonar.login=cd8432698d6588536b374524981c92076bd4b30e"
		  sh "echo '**** SONARQUBE ANALYSIS COMPLETE ******'"	
       } else {
          bat(/"${mvnHome}\bin\mvn" sonar:sonar/)
       }
    }

    stage('Publish to Hygieia') {
       if (isUnix()) {
			sh "echo '**** STARTING BUILD PUBLISH STATUS ******'"					
			hygieiaBuildPublishStep buildStatus: 'Success'
			sh "echo '**** STARTING ARTIFACT PUBLISH ******'"	
			hygieiaArtifactPublishStep artifactDirectory: 'target', artifactGroup: 'com.example.devops', artifactName: '*.war', artifactVersion: ''			
			sh "echo '**** STARTING PUBLISH STEP******'"	
			hygieiaTestPublishStep buildStatus: 'Success', testApplicationName: 'devops-2', testEnvironmentName: 'DEV', testFileNamePattern: 'TEST-*.xml', testResultsDirectory: '/target/surefire-reports/', testType: 'Unit'
			sh "echo '**** STARTING CODE QUALITY PUBLISH STEP******'"	
			hygieiaCodeQualityPublishStep checkstyleFilePattern: '**/*/checkstyle-result.xml', findbugsFilePattern: '**/*/Findbugs.xml', jacocoFilePattern: '**/*/jacoco.xml', junitFilePattern: '**/*/TEST-.*-test.xml', pmdFilePattern: '**/*/PMD.xml'
			sh "echo '**** STARTING SONAR PUBLISH STEP******'"
			hygieiaSonarPublishStep ceQueryIntervalInSeconds: '10', ceQueryMaxAttempts: '30'
			sh "echo '**** STARTING DEPLOY PUBLISH STEP******'"
			hygieiaDeployPublishStep applicationName: 'devops-2', artifactDirectory: 'target', artifactGroup: 'com.example.devops', artifactName: '*.war', artifactVersion: '', buildStatus: 'InProgress', environmentName: 'DEV'
			hygieiaDeployPublishStep applicationName: 'devops-2', artifactDirectory: 'target', artifactGroup: 'com.example.devops', artifactName: '*.war', artifactVersion: '', buildStatus: 'InProgress', environmentName: 'TEST'
			hygieiaDeployPublishStep applicationName: 'devops-2', artifactDirectory: 'target', artifactGroup: 'com.example.devops', artifactName: '*.war', artifactVersion: '', buildStatus: 'InProgress', environmentName: 'PROD'
			sh "echo '**** PUBLISH TO HYGIEIA COMPLETE ******'"
       } else {
			sh "echo '**** STARTING SOMETHING WENT WRONG ******'"	
          //bat(/"${mvnHome}\bin\mvn" sonar:sonar/)
       }
    }

    if(env.BRANCH_NAME == 'master'){
      stage('Validate Build Post Prod Release') {
        if (isUnix()) {
           sh "'${mvnHome}/bin/mvn' clean package"
        } else {
           bat(/"${mvnHome}\bin\mvn" clean package/)
        }
      }

    }

      if(env.BRANCH_NAME == 'develop'){
        stage('Snapshot Build And Upload Artifacts') {
          if (isUnix()) {
             sh "'${mvnHome}/bin/mvn' clean deploy"
          } else {
             bat(/"${mvnHome}\bin\mvn" clean deploy/)
          }
        }

        stage('Deploy') {
           sh 'curl -u jenkins:jenkins -T target/**.war "http://localhost:8080/manager/text/deploy?path=/devops&update=true"'
        }

        stage("Smoke Test"){
           sh "curl --retry-delay 10 --retry 5 http://localhost:8080/devops"
        }

      }

      if(env.BRANCH_NAME ==~ /release.*/){
        pom = readMavenPom file: 'pom.xml'
        artifactVersion = pom.version.replace("-SNAPSHOT", "")
        tagVersion = 'v'+artifactVersion

        stage('Release Build And Upload Artifacts') {
          if (isUnix()) {
             sh "'${mvnHome}/bin/mvn' clean release:clean release:prepare release:perform"
          } else {
             bat(/"${mvnHome}\bin\mvn" clean release:clean release:prepare release:perform/)
          }
        }
         stage('Deploy To Dev') {
            sh 'curl -u jenkins:jenkins -T target/**.war "http://localhost:8080/manager/text/deploy?path=/devops&update=true"'
         }

         stage("Smoke Test Dev"){
             sh "curl --retry-delay 10 --retry 5 http://localhost:8080/devops"
         }

         stage("QA Approval"){
             echo "Job '${env.JOB_NAME}' (${env.BUILD_NUMBER}) is waiting for input. Please go to ${env.BUILD_URL}."
             input 'Approval for QA Deploy?';
         }

         stage("Deploy from Artifactory to QA"){
           retrieveArtifact = 'http://localhost:8081/artifactory/libs-release-local/com/example/devops/' + artifactVersion + '/devops-' + artifactVersion + '.war'
           echo "${tagVersion} with artifact version ${artifactVersion}"
           echo "Deploying war from http://localhost:8081/artifactory/libs-release-local/com/example/devops/${artifactVersion}/devops-${artifactVersion}.war"
           sh 'curl -O ' + retrieveArtifact
           sh 'curl -u jenkins:jenkins -T *.war "http://localhost:7080/manager/text/deploy?path=/devops&update=true"'
         }

      }
  } catch (exception) {
      committerEmail = sh (script: 'git --no-pager show -s --format=\'%ae\'', returnStdout: true).trim()
      emailext(body: '${DEFAULT_CONTENT}', mimeType: 'text/html', replyTo: '$DEFAULT_REPLYTO', subject: '${DEFAULT_SUBJECT}', to: committerEmail)
      throw exception
  }

}
