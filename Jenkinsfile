pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh '''
                echo "Building Docker image..."
                DOCKER_BUILDKIT=0 /usr/local/bin/docker build --no-cache -t hello-world-django-app:${BUILD_NUMBER} .
                /usr/local/bin/docker images | grep hello-world-django-app
                '''
            }
        }
        
        stage('Test') {
            steps {
                sh '''
                echo "Running Django health check..."
                /usr/local/bin/docker run --rm hello-world-django-app:${BUILD_NUMBER} python manage.py check --deploy
                '''
            }
        }
        
        stage('Docker Hub Push') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo "Logging into Docker Hub..."
                    echo $DOCKER_PASS | /usr/local/bin/docker login -u $DOCKER_USER --password-stdin
                    
                    echo "Tagging images..."
                    /usr/local/bin/docker tag hello-world-django-app:${BUILD_NUMBER} $DOCKER_USER/hello-world-django-app:${BUILD_NUMBER}
                    /usr/local/bin/docker tag hello-world-django-app:${BUILD_NUMBER} $DOCKER_USER/hello-world-django-app:latest
                    
                    echo "Pushing to Docker Hub..."
                    /usr/local/bin/docker push $DOCKER_USER/hello-world-django-app:${BUILD_NUMBER}
                    /usr/local/bin/docker push $DOCKER_USER/hello-world-django-app:latest
                    
                    echo "✅ FULL SUCCESS - Docker Hub LIVE!"
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
