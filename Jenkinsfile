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

        stage('Deploy Backend Containers') {
            steps {
                sh '''
                docker network create app-network 2>/dev/null || true
        
                # Force remove backend1 if exists
                docker ps -aq --filter "name=^backend1$" | xargs -r docker rm -f
        
                # Force remove backend2 if exists
                docker ps -aq --filter "name=^backend2$" | xargs -r docker rm -f
        
                docker run -d --name backend1 --network app-network backend-app
                docker run -d --name backend2 --network app-network backend-app
                '''
            }
        }

        stage('Deploy NGINX Load Balancer') {
            steps {
                sh '''
                docker rm -f nginx-lb 2>/dev/null || true

                docker run -d \
                  --name nginx-lb \
                  --network app-network \
                  -p 8081:80 \
                  nginx

                docker cp nginx/default.conf nginx-lb:/etc/nginx/conf.d/default.conf
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
