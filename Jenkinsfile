node {
  def mvnHome
  def pom
  def artifactVersion
  def tagVersion
  def retrieveArtifact

  stage('Prepare') {
    mvnHome = tool 'maven'
  }

  stage('Checkout') {
     checkout scm
  }

  stage('Build') {
     sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean package"
  }

  stage('Unit Test') {
     junit '**/target/surefire-reports/TEST-*.xml'
     archive 'target/*.jar'
  }

  stage('Integration Test') {
    sh "'${mvnHome}/bin/mvn' -Dmaven.test.failure.ignore clean verify"
  }

  stage('Sonar') {
     sh "'${mvnHome}/bin/mvn' sonar:sonar"
  }

  if(env.BRANCH_NAME == 'master'){
    stage('Validate Build Post Prod Release') {
      sh "'${mvnHome}/bin/mvn' clean package"
    }
  }

  if(env.BRANCH_NAME == 'develop'){
    stage('Snapshot Build And Upload Artifacts') {
      sh "'${mvnHome}/bin/mvn' clean deploy"
    }

    stage('Deploy') {
       sh 'curl -u jenkins:jenkins -T target/**.war "http://localhost:8080/manager/text/deploy?path=/devops&update=true"'
    }

    stage("Smoke Test"){
       sh "curl --retry-delay 10 --retry 5 http://localhost:8080/devops"
    }
  }
}