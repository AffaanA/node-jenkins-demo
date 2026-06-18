pipeline {
    agent any
    stages {
        stage('Checkout Source Code') {
            steps {
                // Force a clean workspace to fix local Windows file locks
                cleanWs()
                // Explicitly pull down the repository before Docker steps run
                checkout scm
            }
        }
        stage('Tests') {
            steps {
                 script {
                    docker.image('node:18').inside {
                        echo 'Building..'
                        sh 'npm install'
                        echo 'Testing..'
                        sh 'npm test'
                    }
                 }
            }
        }
        stage('Build and push docker image') {
            steps {
                script {
                    def dockerImage = docker.build("affaana/node-demo:master")
                    docker.withRegistry('', 'demo-docker') {
                        dockerImage.push('master')
                    }
                }
            }
        }
        stage('Deploy to remote docker host') {
            steps {
                script {
                    // Using withCredentials is safer than structural environment blocks
                    withCredentials([usernamePassword(credentialsId: 'demo-docker', passwordVariable: 'DOCKER_PSW', usernameVariable: 'DOCKER_USR')]) {
                        // Added || true to prevent the build from failing if the container doesn't exist yet
                        sh 'docker stop node-demo || true'
                        sh 'docker rm node-demo || true'
                        sh 'docker rmi antonml/node-demo:current || true'
                        
                        sh 'docker pull affaana/node-demo:master'
                        sh 'docker tag affaana/node-demo:master affaana/node-demo:current'
                        sh 'docker run -d --name node-demo -p 80:3000 affaana/node-demo:current'
                    }
                }
            }
        }
    }
}

