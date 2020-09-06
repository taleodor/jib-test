BUILD_START_TIME="0"
RELIZA_VER="0"
RELIZA_FULL_VER="0"
RELIZA_SHORT_VER="0"

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
  // options {
  //  withAWS(credentials:'aws')
  // }
  stages {
    stage('Record Start Time') {
        steps {
            script {
                BUILD_START_TIME = sh(script: 'date -Iseconds', returnStdout: true).trim()
            }
        }
    }
    stage('Run maven') {
      steps {
        echo "Reliza Base Ver = ${RELIZA_VER}"
        echo "Reliza Full Ver = ${RELIZA_FULL_VER}, Reliza Short Ver = ${RELIZA_SHORT_VER}"
        container('maven') {
            withCredentials([usernamePassword(credentialsId: 'da8b0f12-0431-4939-9888-3481b95ab7d1', usernameVariable: 'RELIZA_API_ID', passwordVariable: 'RELIZA_API_KEY')]) {
                script {
                    RELIZA_VER = sh(script: 'docker run --rm relizaio/reliza-go-client -u https://test.relizahub.com getversion -k $RELIZA_API_KEY -i $RELIZA_API_ID -b $GIT_BRANCH --metadata Jenkins --project ebc33386-81e1-42a4-8c69-223b013862a9', returnStdout: true).trim()
                    sh 'echo Reliza Ver in Middle = $RELIZA_VER'
                }
            }
            // sh 'echo "reliza full ver = $RELIZA_FULL_VER"'
            // sh 'echo "reliza short ver = $RELIZA_SHORT_VER"'
            
        //  sh 'apk add openjdk11'
        //  sh 'apk add maven'
        //  sh 'mvn clean compile jib:dockerBuild'
        }
      }
    }
    stage ('Extract Short and Full Versions') {
      environment {
        RELIZA_VER = "${RELIZA_VER}"
      }
      steps {
        container('maven') {
            script {
                RELIZA_FULL_VER = sh(script: 'echo $RELIZA_VER | docker run -i --rm relizaio/jq -r ".version"', returnStdout: true).trim()
                RELIZA_SHORT_VER = sh(script: 'echo $RELIZA_VER | docker run -i --rm relizaio/jq -r ".dockerTagSafeVersion"', returnStdout: true).trim()
            }
        }
      }
    }
    stage('Echo Versions') {
        steps {
            echo "Reliza Base Ver = ${RELIZA_VER}"
            echo "Reliza Full Ver = ${RELIZA_FULL_VER}, Reliza Short Ver = ${RELIZA_SHORT_VER}"
        }
    }
    stage('Build Image') {
        steps {
            container('maven') {
                // sh 'docker ps'
                // sh 'docker run -dp 5000:5000 --restart=always --name registry registry'
                // sh 'docker pull localhost:5000/hw'
                // sh 'docker ps'
                withCredentials([usernamePassword(credentialsId: 'aws', usernameVariable: 'AWS_ACCESS_KEY_ID', passwordVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                    sh '$(docker run --rm -e AWS_ACCESS_KEY_ID=$AWS_ACCESS_KEY_ID -e AWS_SECRET_ACCESS_KEY=$AWS_SECRET_ACCESS_KEY relizaio/awscli ecr get-login --no-include-email --region us-west-1)'
                }
                sh 'apk add openjdk11'
                sh 'apk add maven'
                sh 'mvn clean compile jib:build'
            }
        }
    }
  }
}