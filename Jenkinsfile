pipeline {
    agent any
    environment {
        AWS_REGION = 'eu-north-1'                // <-- Choose your AWS region here
        ECR_REGISTRY = '444062204470.dkr.ecr.eu-north-1.amazonaws.com' // <-- Your ECR registry URL
        ECR_REPOSITORY = 'project004'            // <-- Your ECR repository name
        ECS_CLUSTER = 'django-cluster'            // <-- Your ECS cluster name
        ECS_SERVICE = 'django-service'            // <-- Your ECS service name
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
                    aws ecr get-login-password --region $AWS_REGION | /usr/local/bin/docker login --username AWS --password-stdin $ECR_REGISTRY

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
                    sed -e "s|IMAGE_PLACEHOLDER|$ECR_REGISTRY/$ECR_REPOSITORY:${BUILD_NUMBER}|g" task-definition.json > task-definition-updated.json
                    TASK_REVISION=$(aws ecs register-task-definition --cli-input-json file://task-definition-updated.json --query 'taskDefinition.revision' --output text)
                    aws ecs update-service --cluster $ECS_CLUSTER --service $ECS_SERVICE --task-definition $ECR_REPOSITORY:$TASK_REVISION
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
