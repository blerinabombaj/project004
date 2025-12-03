pipeline {
    agent any
    
    stages {
        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    echo $DOCKER_PASS | /usr/local/bin/docker login -u $DOCKER_USER --password-stdin
                    echo "✅ Docker Hub login successful"
                    '''
                }
            }
        }
        
        stage('Build') {
            steps {
                sh '''
                DOCKER_BUILDKIT=0 /usr/local/bin/docker build --no-cache -t hello-world-django-app:${BUILD_NUMBER} .
                echo "✅ Docker image built: hello-world-django-app:${BUILD_NUMBER}"
                /usr/local/bin/docker images | grep hello-world-django-app
                '''
            }
        }
        
        stage('Health Check') {
            steps {
                sh '''
                /usr/local/bin/docker run --rm hello-world-django-app:${BUILD_NUMBER} python manage.py check --deploy
                echo "✅ Django health check PASSED"
                '''
            }
        }
        
        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'dockerhub', 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh '''
                    /usr/local/bin/docker tag hello-world-django-app:${BUILD_NUMBER} $DOCKER_USER/hello-world-django-app:${BUILD_NUMBER}
                    /usr/local/bin/docker tag hello-world-django-app:${BUILD_NUMBER} $DOCKER_USER/hello-world-django-app:latest
                    /usr/local/bin/docker push $DOCKER_USER/hello-world-django-app:${BUILD_NUMBER}
                    /usr/local/bin/docker push $DOCKER_USER/hello-world-django-app:latest
                    echo "✅ Pushed to Docker Hub: $DOCKER_USER/hello-world-django-app:${BUILD_NUMBER}"
                    '''
                }
            }
        }
    }
    
    post {
        always {
            sh '/usr/local/bin/docker system prune -f || true'
        }
        success {
            echo '🎉 FULL DJANGO CI/CD PIPELINE SUCCESS! 🚀'
        }
        failure {
            echo '❌ Pipeline failed - check logs above'
        }
    }
}
