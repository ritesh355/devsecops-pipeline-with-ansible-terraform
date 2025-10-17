pipeline {
    agent any

    environment {
        AWS_REGION = "us-east-1"
        ECR_REPO = "772954893641.dkr.ecr.us-east-1.amazonaws.com/flask-app"
        IMAGE_TAG = "latest"
        ANSIBLE_DIR = "/home/ritesh/devops-project/ansible"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "📦 Cloning repository..."
                git branch: 'main', url: 'https://github.com/ritesh355/devsecops-pipeline-with-ansible-terraform.git'
            }
        }

        stage('Build Docker Image') {
            steps {
                echo "🐳 Building Docker image..."
                sh """
                    docker build -t $ECR_REPO:$IMAGE_TAG .
                """
            }
        }

        stage('Trivy Security Scan') {
            steps {
                echo "🔍 Running Trivy scan..."
                sh """
                    trivy image --exit-code 0 --severity HIGH,CRITICAL $ECR_REPO:$IMAGE_TAG > trivy-report.txt || true
                    echo "✅ Trivy scan completed. Report saved as trivy-report.txt"
                """
            }
        }

        stage('Login to AWS ECR') {
            steps {
                echo "🔑 Logging into AWS ECR..."
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    sh """
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $ECR_REPO
                    """
                }
            }
        }

        stage('Push Image to ECR') {
            steps {
                echo "📤 Pushing image to ECR..."
                sh """
                    docker push $ECR_REPO:$IMAGE_TAG
                """
            }
        }

        stage('Deploy Using Ansible') {
            steps {
                echo "🚀 Deploying application with Ansible..."
                withCredentials([sshUserPrivateKey(credentialsId: 'ansible-key', keyFileVariable: 'SSH_KEY')]) {
                    sh """
                        cd $ANSIBLE_DIR
                        ansible-playbook -i inventory.ini playbook.yml --limit flask_server --key-file \$SSH_KEY
                    """
                }
            }
        }
    }

    post {
        always {
            echo "🧹 Cleaning up..."
            sh 'docker system prune -f || true'
        }
        success {
            echo "✅ Pipeline executed successfully!"
        }
        failure {
            echo "❌ Pipeline failed. Check logs for details."
        }
    }
}
