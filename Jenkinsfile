pipeline {
    agent any

    environment {
        AWS_DEFAULT_REGION = "us-east-1"
        ECR_REPO = "772954893641.dkr.ecr.us-east-1.amazonaws.com/flask-app"
        IMAGE_TAG = "latest"
        ANSIBLE_DIR = "ansible"
    }

    stages {
        stage('Checkout Code') {
            steps {
                echo "📦 Checking out repository..."
                checkout scm
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh '''
                docker build -t $ECR_REPO:$IMAGE_TAG .
                '''
            }
        }

        stage('Security Scan (Trivy)') {
            steps {
                echo "🔍 Scanning Docker image for vulnerabilities..."
                sh '''
                trivy image --exit-code 0 --severity HIGH,CRITICAL $ECR_REPO:$IMAGE_TAG || true
                '''
            }
        }

        stage('Push Image to AWS ECR') {
            steps {
                echo "☁️ Pushing image to AWS ECR..."
                withCredentials([[
                    $class: 'AmazonWebServicesCredentialsBinding',
                    credentialsId: 'aws-ecr-creds'
                ]]) {
                    sh '''
                    aws ecr get-login-password --region $AWS_DEFAULT_REGION \
                        | docker login --username AWS --password-stdin $ECR_REPO
                    docker push $ECR_REPO:$IMAGE_TAG
                    '''
                }
            }
        }

        stage('Deploy with Ansible') {
            steps {
                echo "🚀 Deploying Flask app via Ansible..."
                withCredentials([sshUserPrivateKey(
                    credentialsId: 'ansible-ec2-key',
                    keyFileVariable: 'EC2_KEY'
                )]) {
                    sh '''
                    cd $ANSIBLE_DIR
                    ansible-playbook -i inventory.ini playbook.yml --limit flask_server --private-key $EC2_KEY
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Deployment successful!"
        }
        failure {
            echo "❌ Deployment failed. Check logs in Jenkins console."
        }
    }
}
