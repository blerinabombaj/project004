pipeline {
    agent any
    environment {
        AWS_REGION = 'eu-north-1'
        ECR_REGISTRY = '444062204470.dkr.ecr.eu-north-1.amazonaws.com'
        ECR_REPOSITORY = 'project004'
        ECS_CLUSTER = 'django-cluster'
        ECS_SERVICE = 'django-service'
        AWS_CLI = '/opt/homebrew/bin/aws'
    }
    stages {
        stage('Build & Test') {
            steps {
                sh '''
                echo "Building Docker image..."
                DOCKER_BUILDKIT=0 /usr/local/bin/docker build --no-cache -t hello-world-django-app:${BUILD_NUMBER} .
                /usr/local/bin/docker images | grep hello-world-django-app
                
                echo "Running Django health check..."
                /usr/local/bin/docker run --rm hello-world-django-app:${BUILD_NUMBER} python manage.py check --deploy
                '''
            }
        }
        stage('ECR Push') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr']]) {
                    sh '''
                    echo "Logging into ECR..."
                    ${AWS_CLI} ecr get-login-password --region $AWS_REGION | /usr/local/bin/docker login --username AWS --password-stdin $ECR_REGISTRY
                    
                    IMAGE_TAG=${BUILD_NUMBER}
                    echo "Tagging images..."
                    /usr/local/bin/docker tag hello-world-django-app:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
                    /usr/local/bin/docker tag hello-world-django-app:$IMAGE_TAG $ECR_REGISTRY/$ECR_REPOSITORY:latest
                    
                    echo "Pushing to ECR..."
                    /usr/local/bin/docker push $ECR_REGISTRY/$ECR_REPOSITORY:$IMAGE_TAG
                    /usr/local/bin/docker push $ECR_REGISTRY/$ECR_REPOSITORY:latest
                    echo "✅ ECR Push Complete!"
                    '''
                }
            }
        }
        stage('ECS Deploy') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-ecr']]) {
                    sh '''
                    echo "Updating task definition..."
                    sed -e "s|IMAGE_PLACEHOLDER|$ECR_REGISTRY/$ECR_REPOSITORY:${BUILD_NUMBER}|g" task-definition.json > task-definition-updated.json
                    cat task-definition-updated.json | grep -E "(family|image)"
                    
                    TASK_REVISION=$(${AWS_CLI} ecs register-task-definition --cli-input-json file://task-definition-updated.json --query "taskDefinition.revision" --output text)
                    TASK_FAMILY=$(${AWS_CLI} ecs register-task-definition --cli-input-json file://task-definition-updated.json --query "taskDefinition.family" --output text)
                    
                    echo "Deploying $TASK_FAMILY:$TASK_REVISION to $ECS_CLUSTER/$ECS_SERVICE"
                    ${AWS_CLI} ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --task-definition "$TASK_FAMILY:$TASK_REVISION"
                    
                    echo "Waiting for rollout..."
                    sleep 45
                    echo "✅ ECS DEPLOY COMPLETE! Check AWS Console."
                    '''
                }
            }
        }
    }
    post {
        always {
            sh '/usr/local/bin/docker system prune -f || true'
        }
    }
}
