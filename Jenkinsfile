pipeline {
    agent any

    environment {
        APPLICATION_NAME = "counterapp"
        PORT = "5000"
        BRANCH_NAME = "${params.BRANCH_NAME}"
        BUILD_TAG = "${params.BRANCH_NAME}_${BUILD_NUMBER}"
        GIT = "https://github.com/dor-a/CounterApp.git"
    }

    stages {

        stage('Clean Workspace') {
            steps {
                cleanWs()
            }
        }

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
                script {
                    sh """
                    git clone -b $BRANCH_NAME $GIT $APPLICATION_NAME
                    cd $APPLICATION_NAME
                    ls
                    docker image build -f Dockerfile -t $APPLICATION_NAME:$BUILD_TAG .
                    """
                }
            }
        }

        stage('Stop Previous Version') {
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

        stage('Deploy Latest Application') {
            steps {
                sh "docker run -d -p $PORT:$PORT --name $APPLICATION_NAME $APPLICATION_NAME:$BUILD_TAG"
            }
        }

stage('Check Application Status') {
    steps {
        script {
            retry(3) {
                sleep 10
                String status = sh(script: "curl -X GET -sLI -w '200' http://localhost:$PORT-o /dev/null", returnStdout: true).trim()

                if (status != "200") {
                    error("Returned status code = $status when checking http://localhost:$PORT")
                }
            }
        }
    }
}

    }
}
