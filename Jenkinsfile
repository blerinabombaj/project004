pipeline {
    agent any
    environment {
        AWS_REGION = 'us-east-1'  // Your region
        ECR_REGISTRY = '123456789012.dkr.ecr.us-east-1.amazonaws.com'  // YOUR_ACCOUNT_ID
        ECR_REPOSITORY = 'hello-world-django-app'
        ECS_CLUSTER = 'django-cluster'
        ECS_SERVICE = 'django-service'
    }
    stages {
        stage('Build & Test') { /* Your existing stages - keep */ }
        stage('ECR Push') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr']]) {
                    sh '''
                    # ECR Login
                    aws ecr get-login-password --region $AWS_REGION | /usr/local/bin/docker login --username AWS --password-stdin $ECR_REGISTRY
                    
                    # Tag & Push
                    IMAGE_TAG=${BUILD_NUMBER}
                    /usr/local/bin/docker tag hello-world-django-app:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
                    /usr/local/bin/docker tag hello-world-django-app:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
                    /usr/local/bin/docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
                    /usr/local/bin/docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest
                    '''
                }
            }
        }
        stage('ECS Deploy') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr']]) {
                    sh '''
                    # Update Task Definition with new image
                    sed -e "s|IMAGE_PLACEHOLDER|$ECR_REGISTRY/$ECR_REPOSITORY:$BUILD_NUMBER|g" task-definition.json > task-definition-updated.json
                    
                    # Register new task definition
                    TASK_REVISION=$(aws ecs register-task-definition --cli-input-json file://task-definition-updated.json --query 'taskDefinition.revision' --output text)
                    
                    # Update service
                    aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --task-definition $ECR_REPOSITORY:$TASK_REVISION
                    '''
                }
            }
        }
    }
}
