pipeline {
	agent any
//   def mvnHome = tool 'MAVEN'
//   env.JAVA_HOME = tool'JDK-11'
	stages {
		stage('Prepare') {
			steps {
         git 'https://github.com/amateus1/devopsbase.git'
		}
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
			tools {
					jdk "JDK-8" // the name you have given the JDK installation using the JDK manager (Global Tool Configuration)
				}
            steps {
              sh "whoami"
              sh 'mvn compile'
            }
		}


   stage('Unit Test') {
		steps {
		sh "echo '**** STARTIN UNIT TEST ******'"
		  sh "java -version" 
			sh "echo '**** STARTING JUNIT TEST ******'"
	//      junit '/bitnami/jenkins/home/workspace/coe-pipeline-1/target/surefire-reports/TEST-*.xml'
			sh "echo '**** COMPLETED JUNIT TEST ******'"
		   archiveArtifacts 'target/devops-7.3.0-SNAPSHOT.war'
			sh "echo '**** COMPLETED ARTIFACT ARCHIVE ******'"
			sh "pwd"
			sh "ls"
			sh "cd target"
			sh "cd target"
			sh "ls"
	//         archiveArtifacts 'target/*.jar'
	//      archive 'target/*.jar'
		  hygieiaDeployPublishStep applicationName: 'simple-maven-project-with-tests', artifactDirectory: '/bitnami/jenkins/home/workspace/coe-pipeline-1/target', artifactGroup: 'com.example.devops', artifactName: '*.war', artifactVersion: '', buildStatus: 'InProgress', environmentName: 'DEV'
			hygieiaDeployPublishStep applicationName: 'simple-maven-project-with-tests', artifactDirectory: '/bitnami/jenkins/home/workspace/coe-pipeline-1/target', artifactGroup: 'com.example', artifactName: '*.war', artifactVersion: '', buildStatus: 'Success', environmentName: 'DEV'   
			hygieiaDeployPublishStep applicationName: 'simple-maven-project-with-tests', artifactDirectory: '/bitnami/jenkins/home/workspace/coe-pipeline-1/target', artifactGroup: 'com.example', artifactName: '*.war', artifactVersion: '', buildStatus: 'Success', environmentName: 'QA'
			hygieiaDeployPublishStep applicationName: 'simple-maven-project-with-tests', artifactDirectory: '/bitnami/jenkins/home/workspace/coe-pipeline-1/target', artifactGroup: 'com.example', artifactName: '*.war', artifactVersion: '', buildStatus: 'Success', environmentName: 'PROD'    
	//      	hygieiaCodeQualityPublishStep checkstyleFilePattern: '**/*/checkstyle-result.xml', findbugsFilePattern: '**/*/Findbugs.xml', jacocoFilePattern: '**/*/jacoco.xml', junitFilePattern: '**/*/TEST-.*-test.xml', pmdFilePattern: '**/*/PMD.xml'
			sh "echo '**** COMPLETED UNIT TEST ******'"
		}
		}
   stage('Integration Test') {
		steps {
//     if (isUnix()) {
        sh "echo 'coe+best2022' | sudo -S mvn -Dmaven.test.failure.ignore clean install"
//     } else {
//        bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean verify/)
//     }
		}
	}
 
  stage('SonarQube Analysis') {
//    def mvn = tool 'Default Maven';
    tools {
        jdk "JDK-11" // the name you have given the JDK installation using the JDK manager (Global Tool Configuration)
    }
	environment {
        scannerHome = tool 'SonarQube Scanner' // the name you have given the Sonar Scanner (Global Tool Configuration)
    }
    steps {
		sh "echo '**** STARTING SONAR TEST ******'"
        withSonarQubeEnv(installationName: 'SonarQube') {
            sh 'mvn sonar:sonar'
        }	
	
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
       steps {
	   sh 'curl -u jenkins:jenkins -T target/**.war "http://mep-tomcat.eastus.cloudapp.azure.com:/manager/text/deploy?path=/develop-pipeline&update=true"'
     	hygieiaArtifactPublishStep artifactDirectory: '/develop-pipeline/target', artifactGroup: 'com.example.devops', artifactName: '*war', artifactVersion: '3.0.0'
		}
	}
   stage('Deploy-QA') {
		steps {
// 	   sh 'curl -u jenkins:jenkins -T target/**.war "http://coe-tomcatqa3.eastus.cloudapp.azure.com/manager/text/deploy?path=/develop-pipeline&update=true"'
     hygieiaArtifactPublishStep artifactDirectory: '/develop-pipeline/target', artifactGroup: 'com.example.devops', artifactName: '*war', artifactVersion: '3.0.0'
		}
	}
 	stage('Deploy-PROD') {
		steps {
// 	   sh 'curl -u jenkins:jenkins -T target/**.war "http://mep-tomcat-prod.eastus.cloudapp.azure.com/manager/text/deploy?path=/develop-pipeline&update=true"'
     	hygieiaArtifactPublishStep artifactDirectory: '/develop-pipeline/target', artifactGroup: 'com.example.devops', artifactName: '*war', artifactVersion: '3.0.0'
		}
	}
   stage("Smoke Test-PROD"){
       steps {
	   sh "curl --retry-delay 10 --retry 5 http://mep-tomcat.eastus.cloudapp.azure.com/develop-pipeline"
 //      sh "curl --retry-delay 10 --retry 5 http://coe-tomcatqa3.eastus.cloudapp.azure.com/develop-pipeline/"
 //      sh "curl --retry-delay 10 --retry 5 http://mep-tomcat-prod.eastus.cloudapp.azure.com/develop-pipeline/"
		}
	}
}
}
