node {
   
   def mvnHome = tool 'MAVEN'
   env.JAVA_HOME = tool'JAVA'

   stage('Prepare') {

      //      credentialsID: 'GITHUB-DEPLOY'
//      git fetch --tags --progress https://github.com/amateus1/devopsbase.git
//      git url: 'https://git@github.com:amateus1/devopsbase.git', branch: 'develop'
         git (url: 'https://github.com/amateus1/devopsbase.git', credentialsID: 'aee06964-d162-4114-b8fb-9d622b7e8389', branch: 'develop')

   }
   stage('Build') {
      if (isUnix()) {
//         sh "'/usr/bin/mvn' -Dmaven.test.failure.ignore clean package"
           sh "mvn -Dmaven.test.failure.ignore clean package"
//         sh "/var/lib/jenkins/workspace/develop-pipeline/mvn -version"
//         sh "mvn -Dmaven.test.failure.ignore clean package"
           hygieiaBuildPublishStep buildStatus: 'Success'
  
    } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
      }
   }
   stage('Unit Test') {
      junit '**/target/surefire-reports/TEST-*.xml'
       archive 'target/*.war'
//      archive 'target/*.jar'
//      hygieiaDeployPublishStep applicationName: 'devops', artifactDirectory: '/target', artifactGroup: 'com.example.devops', artifactName: '*.jar', artifactVersion: '', buildStatus: 'InProgress', environmentName: 'DEV'
        hygieiaCodeQualityPublishStep checkstyleFilePattern: '**/*/checkstyle-result.xml', findbugsFilePattern: '**/*/Findbugs.xml', jacocoFilePattern: '**/*/jacoco.xml', junitFilePattern: '**/*/TEST-.*-test.xml', pmdFilePattern: '**/*/PMD.xml'
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
         sh "'${mvnHome}/bin/mvn' sonar:sonar -Dsonar.projectKey=develop-pipeline   -Dsonar.host.url=http://mep-sonar.eastus.cloudapp.azure.com   -Dsonar.login=ef026f77b563ee37ea01bb630b4dc2701ce4a306"
         // hygieiaSonarPublishStep ceQueryIntervalInSeconds: '10', ceQueryMaxAttempts: '30'
      } else {
         bat(/"${mvnHome}\bin\mvn" sonar:sonar/)
      }
   }
   stage('Deploy') {
       sh 'curl -u jenkins:jenkins -T target/**.war "http://mep-tomcat.eastus.cloudapp.azure.com:8080/manager/text/deploy?path=/develop-pipeline&update=true"'
      hygieiaDeployPublishStep applicationName: 'develop-pipeline', artifactDirectory: 'develop-pipeline/target', artifactGroup: 'com.example.develop-pipeline', artifactName: '1.5.0-SNAPSHOT', artifactVersion: '1.5.0', buildStatus: 'Success', environmentName: 'DEV'   
     hygieiaArtifactPublishStep artifactDirectory: '/develop-pipeline/target', artifactGroup: 'com.example.develop-pipeline', artifactName: '1.5.0-SNAPSHOT', artifactVersion: '1.5.0'
   
   }
   stage("Smoke Test"){
       sh "curl --retry-delay 10 --retry 5 http://mep-tomcat.eastus.cloudapp.azure.com:8080/develop-pipeline"
   }
}
