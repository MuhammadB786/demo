pipeline {
  agent any
  options { timestamps(); buildDiscarder(logRotator(numToKeepStr: '15')) }
  triggers { pollSCM('H/5 * * * *') }
  tools { maven 'Maven 3.8.x+'; jdk 'JDK 17' }
  stages {
    stage('Checkout') { steps { checkout scm } }
    stage('Build (Fat JAR)') {
      steps { sh 'mvn -B -DskipTests clean package' }
      post { success { archiveArtifacts artifacts: 'target/*.jar', fingerprint: true } }
    }
    stage('Unit Tests') {
      steps { sh 'mvn -B test' }
      post { always { junit 'target/surefire-reports/*.xml' } }
    }
  }
  post {
    success { echo 'Build OK' }
    failure { echo 'Build failed' }
  }
}
