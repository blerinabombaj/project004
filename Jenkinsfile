pipeline {
    agent any
    
    stages {
        stage('Build') {
            steps {
                sh '/usr/local/bin/docker build -t hello-world-django-app:${BUILD_NUMBER} .'
                sh '/usr/local/bin/docker images | grep hello-world-django-app'
            }
        }
        stage('Health Check') {
            steps {
                sh '/usr/local/bin/docker run --rm hello-world-django-app:${BUILD_NUMBER} python manage.py check --deploy'
            }
        }
        stage('Deploy') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'docker-hub-credentials', 
                                                 usernameVariable: 'DOCKER_USER', 
                                                 passwordVariable: 'DOCKER_PASS')]) {
                    sh 'echo $DOCKER_PASS | /usr/local/bin/docker login -u $DOCKER_USER --password-stdin'
                    sh '/usr/local/bin/docker tag hello-world-django-app:${BUILD_NUMBER} $DOCKER_USER/hello-world-django-app:${BUILD_NUMBER}'
                    sh '/usr/local/bin/docker tag hello-world-django-app:${BUILD_NUMBER} $DOCKER_USER/hello-world-django-app:latest'
                    sh '/usr/local/bin/docker push $DOCKER_USER/hello-world-django-app:${BUILD_NUMBER}'
                    sh '/usr/local/bin/docker push $DOCKER_USER/hello-world-django-app:latest'
                }
            }
        }
    }
}
