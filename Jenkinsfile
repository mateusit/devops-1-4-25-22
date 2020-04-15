node {
   def mvnHome
   stage('Prepare') {
//      credentialsID: 'GITHUB-DEPLOY'
//      git fetch --tags --progress https://github.com/amateus1/devopsbase.git
//      git url: 'https://git@github.com:amateus1/devopsbase.git', branch: 'develop'
         git (url: 'https://github.com/amateus1/devopsbase.git', credentialsID: 'aee06964-d162-4114-b8fb-9d622b7e8389', branch: 'develop')
      mvnHome = tool 'MAVEN'
   }
   stage('Build') {
      if (isUnix()) {
         sh "'/usr/bin/mvn' -Dmaven.test.failure.ignore clean package"
      } else {
         bat(/"${mvnHome}\bin\mvn" -Dmaven.test.failure.ignore clean package/)
      }
   }
   stage('Unit Test') {
      junit '**/target/surefire-reports/TEST-*.xml'
      archive 'target/*.jar'
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
         sh "'${mvnHome}/bin/mvn' sonar:sonar"
      } else {
         bat(/"${mvnHome}\bin\mvn" sonar:sonar/)
      }
   }
   stage('Deploy') {
       sh 'curl -u jenkins:jenkins -T target/**.war "http://mep-tomcat.eastus.cloudapp.azure.com:8080/manager/text/deploy?path=/devops&update=true"'
   }
   stage("Smoke Test"){
       sh "curl --retry-delay 10 --retry 5 http://mep-tomcat.eastus.cloudapp.azure.com:8080/devops"
   }
}
