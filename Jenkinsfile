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
    volumeMounts:
    - mountPath: /var/run/docker.sock
      name: dockersock
    tty: true
  volumes:
  - name: dockersock
    hostPath:
      path: /var/run/docker.sock
      type: File
"""
    }
  }
  stages {
    stage('Run maven') {
      steps {
        container('maven') {
            withCredentials([usernamePassword(credentialsId: 'da8b0f12-0431-4939-9888-3481b95ab7d1', usernameVariable: 'RELIZA_API_ID', passwordVariable: 'RELIZA_API_KEY')]) {
                sh 'reliza_ver=$(docker run --rm relizaio/reliza-go-client -u https://test.relizahub.com getversion -k $RELIZA_API_KEY -i $RELIZA_API_ID -b $GITHUB_REF --metadata Jenkins --project ebc33386-81e1-42a4-8c69-223b013862a9); echo $reliza_ver'
            }
        //  sh 'apk add openjdk11'
        //  sh 'apk add maven'
        //  sh 'mvn clean compile jib:dockerBuild'
        }
      }
    }
  }
}