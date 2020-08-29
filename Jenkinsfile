pipeline {
  agent {
    kubernetes {
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    some-label: some-label-value
spec:
  containers:
  - name: maven
    image: docker:19.03.12-dind
    command:
    - cat
    tty: true
"""
    }
  }
  stages {
    stage('Run maven') {
      steps {
        container('maven') {
          sh 'apk add openjdk11'
          sh 'apk add maven'
          sh 'mvn clean compile jib:dockerBuild'
        }
      }
    }
  }
}