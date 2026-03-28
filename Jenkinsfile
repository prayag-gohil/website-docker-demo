pipeline {  
    agent any  

    environment {  
        AWS_REGION = 'ap-south-1'  
        ECR_REPO = 'website-docker-demo'  
        AWS_ACCOUNT_ID = '043207749565'  
        IMAGE_TAG = "${env.BUILD_NUMBER}"  
        IMAGE_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:${IMAGE_TAG}"  
        LATEST_URI = "${AWS_ACCOUNT_ID}.dkr.ecr.${AWS_REGION}.amazonaws.com/${ECR_REPO}:latest"  
        DEPLOY_SERVER = '3.110.29.70'  
    }  

    stages {  

        stage('Checkout') {  
            steps {  
                git branch: 'main', url: 'https://github.com/prayag-gohil/website-docker-demo'  
            }  
        }  

        stage('Build Docker Image') {  
            steps {  
                sh 'docker build -t website-docker-demo .'  
            }  
        }  

        stage('Tag Docker Image') {  
            steps {  
                sh '''  
                    docker tag website-docker-demo:latest $IMAGE_URI  
                    docker tag website-docker-demo:latest $LATEST_URI  
                '''  
            }  
        }  

        stage('Login to ECR') {  
            steps {  
                sh '''  
                    aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com  
                '''  
            }  
        }  

        stage('Push Image to ECR') {  
            steps {  
                sh '''  
                    docker push $IMAGE_URI  
                    docker push $LATEST_URI  
                '''  
            }  
        }  

       stage('Deploy to EC2') {
    steps {
        sshagent(['ubuntu']) {
            sh '''
ssh -o StrictHostKeyChecking=no ubuntu@3.110.29.70 "
# Install AWS CLI if not installed
if ! command -v aws > /dev/null; then
  sudo apt update && sudo apt install -y awscli
fi

# Install Docker if not installed
if ! command -v docker > /dev/null; then
  sudo apt update && sudo apt install -y docker.io
  sudo systemctl start docker
fi

# Login to ECR
aws ecr get-login-password --region ap-south-1 | docker login --username AWS --password-stdin 043207749565.dkr.ecr.ap-south-1.amazonaws.com

# Stop old container
docker stop website-demo || true
docker rm website-demo || true

# Pull latest image
docker pull 043207749565.dkr.ecr.ap-south-1.amazonaws.com/website-docker-demo:latest

# Run container
docker run -d -p 80:80 --name website-demo 043207749565.dkr.ecr.ap-south-1.amazonaws.com/website-docker-demo:latest
"
'''
        }
    }
}

    }  

    post {  
        success {  
            echo 'Website deployed successfully 🚀'  
        }  
        failure {  
            echo 'Pipeline failed '  
        }  
    }  
}
