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
                sshagent(['deploy-ec2-key']) {  
                    sh '''  
                        ssh -o StrictHostKeyChecking=no ubuntu@$DEPLOY_SERVER "
                        aws ecr get-login-password --region $AWS_REGION | docker login --username AWS --password-stdin $AWS_ACCOUNT_ID.dkr.ecr.$AWS_REGION.amazonaws.com &&
                        docker pull $LATEST_URI &&
                        docker stop website-demo || true &&
                        docker rm website-demo || true &&
                        docker run -d --name website-demo -p 80:80 $LATEST_URI
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
