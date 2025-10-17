pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "772954893641.dkr.ecr.us-east-1.amazonaws.com/flask-app"
        IMAGE_TAG = "latest"
    }

    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', credentialsId: 'github-creds', url: 'https://github.com/<your-username>/<your-flask-repo>.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    sh "docker build -t ${ECR_REPO}:${IMAGE_TAG} ."
                }
            }
        }

        stage('Push to ECR') {
            steps {
                withAWS(region: "${AWS_REGION}", credentials: 'aws-ecr-creds') {
                    script {
                        sh """
                        aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REPO}
                        docker push ${ECR_REPO}:${IMAGE_TAG}
                        """
                    }
                }
            }
        }

        stage('Deploy on Flask EC2') {
            steps {
                sshagent(['ec2-key']) {
                    sh '''
                    ansible-playbook -i inventory flask_deploy.yml
                    '''
                }
            }
        }
    }
}

