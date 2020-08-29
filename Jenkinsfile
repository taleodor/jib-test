pipeline {
  stages {
    stage('Build Jib Test') {
      steps {
        sh 'mvn clean compile jib:dockerBuild'
      }
    }
  }
}