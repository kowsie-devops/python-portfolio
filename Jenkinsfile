pipeline {
    agent any

    environment {
        DOCKERHUB = "kowsie/python-portfolio"
        EC2_IP = "15.207.110.120"
    }

    stages {

        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/kowsie-devops/python-portfolio.git'
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                docker run --rm \
                     -v $PWD:/app \
                     -w /app \
                     python:3.9 \
                     pip install -r app/requirements.txt
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${DOCKERHUB}:${BUILD_NUMBER} ."
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([string(credentialsId: 'dockerhub-creds', Variable: 'PASS' )]) {
                    sh "echo $PASS | docker login -u kowsie --password-stdin"
                    sh "docker push ${DOCKERHUB}:${BUILD_NUMBER}"
                }
            }
        }

        stage('Deploy to EC2') {
            steps {
                sshagent(['ec2-ssh']) {
                    sh '''
                    ssh -o StrictHostKeyChecking=no ubuntu@$EC2_IP "
                    docker pull ${DOCKERHUB}:${BUILD_NUMBER} &&
                    docker stop portfolio || true &&
                    docker rm portfolio || true &&
                    docker run -d -p 80:5000 --name portfolio ${DOCKERHUB}:${BUILD_NUMBER}
                    "
                    '''
                }
            }
        }
    }
}
