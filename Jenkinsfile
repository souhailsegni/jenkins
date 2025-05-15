pipeline {
    agent any

    environment {
        IMAGE_NAME = 'souhail1920/flask-jenkins'   // change to your Docker Hub username/repo
    }

    stages {
        stage('Clone Repo') {
            steps {
                git 'https://github.com/souhailsegni/jenkins.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh 'docker build -t $IMAGE_NAME .'
                }
            }
        }

        stage('Login to Docker Hub') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub-creds', usernameVariable: 'DOCKER_USER', passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin'
                }
            }
        }

        stage('Push Docker Image') {
            steps {
                sh 'docker push $IMAGE_NAME'
            }
        }

        stage('Deploy (Optional)') {
            steps {
                sh 'docker run -d -p 5000:5000 --name flask-app $IMAGE_NAME'
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution complete.'
        }
    }
}
