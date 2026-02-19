pipeline {
    agent any

    stages {

        stage('Build Backend Image') {
            steps {
                sh '''
                docker rmi -f backend-app || true
                docker build -t backend-app backend
                '''
            }
        }

        stage('Deploy Backend & Network') {
            steps {
                sh '''
                # Remove old containers
                docker rm -f backend1 backend2 nginx-lb || true

                # Remove old network
                docker network rm app-network || true

                # Create fresh network
                docker network create app-network

                # Start backend containers
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app

                # Wait for backend to be ready
                sleep 3
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                # Start nginx container
                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 8081:80 \
                  nginx

                # Wait before copying config
                sleep 2

                # Copy nginx config
                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf

                # Reload nginx
                docker exec nginx-lb nginx -s reload
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline executed successfully. NGINX load balancer is running.'
        }
        failure {
            echo 'Pipeline failed. Check console logs for errors.'
        }
    }
}
