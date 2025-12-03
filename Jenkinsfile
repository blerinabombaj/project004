pipeline {
    agent any
    
    stages {
        stage('Debug Workspace') {
            steps {
                sh '''
                echo "=== WORKSPACE FILES ==="
                pwd
                ls -la
                cat Dockerfile | head -5
                echo "=== END FILES ==="
                '''
            }
        }
        
        stage('Test Build ONLY') {
            steps {
                sh '''
                echo "=== BUILDING IMAGE ==="
                DOCKER_BUILDKIT=0 /usr/local/bin/docker build \
                  --no-cache --pull=false \
                  -t hello-world-django-app:test .
                echo "✅ BUILD SUCCESS!"
                /usr/local/bin/docker images | grep hello-world-django-app
                '''
            }
        }
        
        stage('Test Health Check') {
            steps {
                sh '''
                /usr/local/bin/docker run --rm hello-world-django-app:test python manage.py check --deploy
                echo "✅ HEALTH CHECK PASSED!"
                '''
            }
        }
    }
    
    post {
        success {
            echo '🎉 BUILD + HEALTH CHECK WORKS! Ready for Docker Hub push 🚀'
        }
        always {
            sh '/usr/local/bin/docker rmi hello-world-django-app:test || true'
        }
    }
}
