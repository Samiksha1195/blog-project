pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }

    environment {
        DOCKERHUB_USER = 'samiksha1195'
        FRONTEND_DIR = 'frontend'
        BACKEND_DIR = 'backend'
        APP_HOST = '10-0-2-161'
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Samiksha1195/blog-project.git'
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                  docker build -t $DOCKERHUB_USER/blog-frontend:latest "$FRONTEND_DIR"
                  docker build -t $DOCKERHUB_USER/blog-backend:latest "$BACKEND_DIR"
                '''
            }
        }

        stage('Push Images') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 'dockerhub-creds',
                        usernameVariable: 'DH_USER',
                        passwordVariable: 'DH_TOKEN'
                    )
                ]) {
                    sh '''
                      echo "$DH_TOKEN" | docker login \
                        -u "$DH_USER" --password-stdin

                      docker push $DOCKERHUB_USER/blog-frontend:latest
                      docker push $DOCKERHUB_USER/blog-backend:latest

                      docker logout
                    '''
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['app-ssh-key']) {
                    sh '''
                      ssh -o StrictHostKeyChecking=accept-new \
                        ec2-user@$APP_HOST \
                        'sudo docker pull samiksha1195/blog-frontend:latest &&
                         sudo docker pull samiksha1195/blog-backend:latest &&
                         sudo docker rm -f blog-frontend blog-backend || true'
                    '''
                }

                sshagent(credentials: ['app-ssh-key']) {
                    sh '''
                      ssh ec2-user@$APP_HOST \
                        'sudo docker run -d --restart unless-stopped \
                         --name blog-frontend -p 80:80 \
                         samiksha1195/blog-frontend:latest'

                      ssh ec2-user@$APP_HOST \
                        'sudo docker run -d --restart unless-stopped \
                         --name blog-backend -p 8081:80 \
                         samiksha1195/blog-backend:latest'
                    '''
                }
            }
        }
    }

    post {
        success {
            echo 'Blogging platform deployment succeeded.'
        }
        failure {
            echo 'Deployment failed. Review the console output.'
        }
    }
}
