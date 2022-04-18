node {
   
   def mvnHome = tool 'MAVEN'
   env.JAVA_HOME = tool'JDK-11'

   stage('Prepare') {

      //      credentialsID: 'GITHUB-DEPLOY'
//      git fetch --tags --progress https://github.com/amateus1/devopsbase.git
//      git url: 'https://git@github.com:amateus1/devopsbase.git', branch: 'develop'
         git (url: 'https://github.com/amateus1/devopsbase.git', credentialsID: 'aee06964-d162-4114-b8fb-9d622b7e8389', branch: 'develop')

   }
//   stage('Build') {
//      if (isUnix()) {
//         sh "'/usr/bin/mvn' -Dmaven.test.failure.ignore clean package"
//           sh "mvn -Dmaven.test.failure.ignore clean package"
//         sh "/var/lib/jenkins/workspace/develop-pipeline/mvn -version"
//         sh "mvn -Dmaven.test.failure.ignore clean package"
//           hygieiaBuildPublishStep buildStatus: 'Success'
  
//    } else {
//         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
//      }
// 	}
       stage('Build') {
//            steps {
              sh "whoami"
              sh "echo 'coe+best2022' | sudo -S mvn -Dmaven.test.failure.ignore=true clean install package"
//            }
		}


   stage('Unit Test') {
	  sh "echo '**** STARTIN UNIT TEST ******'"
	  sh "java -version" 
      junit '/bitnami/jenkins/home/workspace/coe-pipeline-1/target/surefire-reports/TEST-*.xml'
//		sh "echo '**** COMPLETED UNIT TEST ******'"
       archiveArtifacts '/bitnami/jenkins/home/workspace/coe-pipeline-1/target/*.war'
	   	sh "echo '**** COMPLETED ARTIFACT ARCHIVE ******'"
		sh "pwd"
		ssh "ls"
		sh "cd target"
		sh "cd target"
		sh "ls"
//         archiveArtifacts 'target/*.jar'
//      archive 'target/*.jar'
      hygieiaDeployPublishStep applicationName: 'simple-maven-project-with-tests', artifactDirectory: '/target', artifactGroup: 'com.example.devops', artifactName: '*.war', artifactVersion: '', buildStatus: 'InProgress', environmentName: 'DEV'
        hygieiaDeployPublishStep applicationName: 'simple-maven-project-with-tests', artifactDirectory: '/bitnami/jenkins/home/workspace/coe-pipeline-1/target', artifactGroup: 'com.example', artifactName: '*.war', artifactVersion: '', buildStatus: 'Success', environmentName: 'DEV'   
        hygieiaDeployPublishStep applicationName: 'simple-maven-project-with-tests', artifactDirectory: '/bitnami/jenkins/home/workspace/coe-pipeline-1/target', artifactGroup: 'com.example', artifactName: '*.war', artifactVersion: '', buildStatus: 'Success', environmentName: 'QA'
        hygieiaDeployPublishStep applicationName: 'simple-maven-project-with-tests', artifactDirectory: '/bitnami/jenkins/home/workspace/coe-pipeline-1/target', artifactGroup: 'com.example', artifactName: '*.war', artifactVersion: '', buildStatus: 'Success', environmentName: 'PROD'    
//      	hygieiaCodeQualityPublishStep checkstyleFilePattern: '**/*/checkstyle-result.xml', findbugsFilePattern: '**/*/Findbugs.xml', jacocoFilePattern: '**/*/jacoco.xml', junitFilePattern: '**/*/TEST-.*-test.xml', pmdFilePattern: '**/*/PMD.xml'
		sh "echo '**** COMPLETED UNIT TEST ******'"
   }
   stage('Integration Test') {
//     if (isUnix()) {
        sh "echo 'coe+best2022' | sudo -S mvn -Dmaven.test.failure.ignore clean install"
//     } else {
//        bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean verify/)
//     }
   }

 
  stage('SonarQube Analysis') {
//    def mvn = tool 'Default Maven';
    sh "echo '**** STARTING SONAR TEST ******'"
	withSonarQubeEnv() {
      sh "echo 'coe+best2022' | sudo -S mvn clean install sonar:sonar -Dsonar.projectKey=simple-maven-project-with-tests -Dsonar.host.url=http://mep-sonar.eastus2.cloudapp.azure.com  -Dsonar.login=fe5b9d9f8a95064ec4a4547c850700dd78c1b038" 
      sh "echo '**** FINISHED SONAR TEST ******'"
	  }
    }

//   stage('Sonar') {
//      if (isUnix()) {
//           sh "'${mvnHome}/bin/mvn' sonar:sonar -Dsonar.projectKey=simple-maven-project-with-tests   -Dsonar.host.url=http://mep-sonar.eastus2.cloudapp.azure.com  -Dsonar.login=ef026f77b563ee37ea01bb630b4dc2701ce4a306" -X
// 		    // sh "'${mvnHome}/bin/mvn' sonar:sonar -Dsonar.projectKey=simple-maven-project-with-tests   -Dsonar.host.url= http://54.227.225.164:9000 -Dsonar.login= f68d0b7ae24f4963c23d2b65bd7ae17fc0f800e1" -X
//         // hygieiaSonarPublishStep ceQueryIntervalInSeconds: '10', ceQueryMaxAttempts: '30'
//      } else {
//         bat(/"${mvnHome}\bin\mvn" sonar:sonar/)
//      }
//   }
   stage('Deploy-DEV') {
       sh 'curl -u jenkins:jenkins -T target/**.war "http://mep-tomcat.eastus.cloudapp.azure.com:/manager/text/deploy?path=/develop-pipeline&update=true"'
     	hygieiaArtifactPublishStep artifactDirectory: '/develop-pipeline/target', artifactGroup: 'com.example.devops', artifactName: '*war', artifactVersion: '3.0.0'
 	}
   stage('Deploy-QA') {
// 	   sh 'curl -u jenkins:jenkins -T target/**.war "http://coe-tomcatqa3.eastus.cloudapp.azure.com/manager/text/deploy?path=/develop-pipeline&update=true"'
     hygieiaArtifactPublishStep artifactDirectory: '/develop-pipeline/target', artifactGroup: 'com.example.devops', artifactName: '*war', artifactVersion: '3.0.0'
 	}
 	stage('Deploy-PROD') {
// 	   sh 'curl -u jenkins:jenkins -T target/**.war "http://mep-tomcat-prod.eastus.cloudapp.azure.com/manager/text/deploy?path=/develop-pipeline&update=true"'
     	hygieiaArtifactPublishStep artifactDirectory: '/develop-pipeline/target', artifactGroup: 'com.example.devops', artifactName: '*war', artifactVersion: '3.0.0'
    }
   stage("Smoke Test-PROD"){
       sh "curl --retry-delay 10 --retry 5 http://mep-tomcat.eastus.cloudapp.azure.com/develop-pipeline"
 //      sh "curl --retry-delay 10 --retry 5 http://coe-tomcatqa3.eastus.cloudapp.azure.com/develop-pipeline/"
 //      sh "curl --retry-delay 10 --retry 5 http://mep-tomcat-prod.eastus.cloudapp.azure.com/develop-pipeline/"
       
   }
}
