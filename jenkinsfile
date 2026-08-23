pipeline {

    agent any

    environment {

        IMAGE = "dockerhubusername/blogging-platform"

    }

    stages {

        stage('Clone GitHub') {

            steps {

                git branch: 'main',
                    url: 'https://github.com/Samiksha1195/blog-project.git'

            }

        }

        stage('Build Docker Image') {

            steps {

                sh 'docker build -t $IMAGE:latest .'

            }

        }

        stage('Login Docker Hub') {

            steps {

                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred',

                usernameVariable: 'USER',

                passwordVariable: 'PASS')]) {

                sh 'echo $PASS | docker login -u $USER --password-stdin'

                }

            }

        }

        stage('Push Docker Image') {

            steps {

                sh 'docker push $IMAGE:latest'

            }

        }

        stage('Deploy Container') {

            steps {

                sh '''

                docker rm -f blog-container || true

                docker pull $IMAGE:latest

                docker run -d --name blog-container -p 80:80 $IMAGE:latest

                '''

            }

        }

    }

}
