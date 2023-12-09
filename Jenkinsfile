pipeline {
  agent any {
  }
  options {
        ansiColor('xterm')
  }

  environment {

    APPLICATION_NAME = "CounterApp"
    PORT = "5000"
    BRANCH_NAME = "${params.BRANCH_NAME}"
    BUILD_TAG = "${params.BRANCH_NAME}_${BUILD_NUMBER}"
    GIT = "https://github.com/dor-a/CounterApp.git"
  }
  stages {
    stage('Preping ENV') {
      steps {
        script {

        println "BRANCH_NAME = ${params.BRANCH_NAME}"
        println "DOCKER-IMAGE = ${APPLICATION_NAME}:${BUILD_TAG}"
      }
     }
    }

    stage('Build Images And Create The Tag') {
      steps {
          echo "Please Note that Build is happen inside the Dockerfile"
           {
            sh "git clone -b $BRANCH_NAME $GIT $APPLICATION_NAME"
            sh "cd $APPLICATION_NAME"
            sh "ls"
            sh "docker image build -f Dockerfile -t $APPLICATION_NAME:$BUILD_TAG ."
          }
      }
    }

    stage('Stop Previous Version ') {
      steps {
        script {
            def runningContainers = sh(script: "docker ps -q --filter name=$APPLICATION_NAME", returnStdout: true).trim()
            if (runningContainers) {
                sh "docker stop $APPLICATION_NAME"
                sh "docker rm $APPLICATION_NAME"
            } else {
                echo "Container '$APPLICATION_NAME' is not running."
            }
        }
      }
    }
    stage('Deploy Latest Application ') {
      steps {
          sh "docker run -d -p $PORT:$PORT --name $APPLICATION_NAME $APPLICATION_NAME:$BUILD_TAG "
      }
    }

    stage('Testing Application') {
      steps {
          sh "finished"
      }
    }
  }
}
