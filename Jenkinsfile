pipeline {
    agent any
    environment {
        DOCKER_ID = 'dstdockerhub'
        DOCKER_IMAGE = 'datascientestapi'
        DOCKER_TAG = "v.${BUILD_ID}.0"
        DOCKERHUB_CREDENTIALS = credentials('DOCKER_HUB_PASS')
    }
    stages {
        stage('Setup Python') {
            steps {
                sh '''
                python3 -m venv venv
                . venv/bin/activate
                pip install -r requirements.txt
                '''
            }
        }
        stage('Building') {
            steps {
                sh 'pip install -r requirements.txt'
            }
        }
        stage('Testing') {
            steps {
                sh '''
                . venv/bin/activate
                python -m unittest
                '''
            }
        }
        stage('Deploying') {
            steps {
                script {
                    deployDocker()
                }
            }
        }
        stage('User Acceptance') {
            steps {
                input message: 'Proceed to push to main', ok: 'Yes'
            }
        }
        stage('Pushing and Merging') {
            steps {
                script {
                    parallel(
                        "Pushing Image": { pushDockerImage() },
                        "Merging": { mergeChanges() }
                    )
                }
            }
        }
    }
    post {
        always {
            sh 'docker logout'
        }
    }
}

void deployDocker() {
    sh '''
    docker ps -a --format '{{.Names}}' | grep -w jenkins && docker rm -f jenkins || true
    docker build -t $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG .
    docker run -d -p 8000:8000 --name jenkins $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG
    '''
}

void pushDockerImage() {
    withEnv(['DOCKERHUB_CREDENTIALS']) {
        sh 'echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin'
        sh 'docker push $DOCKER_ID/$DOCKER_IMAGE:$DOCKER_TAG'
    }
}

void mergeChanges() {
    echo 'Merging done'
}
