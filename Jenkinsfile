pipeline {
    agent any
    
    environment {
        DOCKER_API_VERSION = '1.43'
        
        // --- CẤU HÌNH CHUẨN ---
        GIT_REPO_URL    = 'https://github.com/cnguyenmanh26/vietmythluminarts-api.git'
        GIT_BRANCH      = 'main'
        
        CONTAINER_NAME  = 'backend'             // Container MỚI (Jenkins quản lý)
        OLD_CONTAINER   = 'home-backend-1'      // Container CŨ (Docker Compose quản lý)
        
        IMAGE_NAME      = 'vietmyth-backend-image'
        NETWORK_NAME    = 'home_default'
        ENV_FILE_PATH   = '/home/back-end/.env'
        APP_PORT        = '5000'
        NGINX_CONTAINER = 'home-nginx-1'
    }

    stages {
        stage('1. Checkout') {
            steps {
                git branch: "${GIT_BRANCH}", url: "${GIT_REPO_URL}"
            }
        }

        stage('2. Build') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:latest ."
            }
        }

        stage('3. Deploy (Dọn sạch sẽ)') {
            steps {
                script {
                    // DÙNG || true ĐỂ KHÔNG BÁO LỖI NẾU KHÔNG TÌM THẤY
                    
                    // 1. Diệt container cũ (nếu nó vô tình sống lại)
                    sh "docker stop ${OLD_CONTAINER} || true"
                    sh "docker rm ${OLD_CONTAINER} || true"
                    
                    // 2. Diệt container hiện tại (để cập nhật code mới)
                    sh "docker stop ${CONTAINER_NAME} || true"
                    sh "docker rm ${CONTAINER_NAME} || true"
                    
                    // 3. Chạy container mới
                    sh """
                        docker run -d \
                        --name ${CONTAINER_NAME} \
                        --network ${NETWORK_NAME} \
                        --restart always \
                        --env-file ${ENV_FILE_PATH} \
                        -e NODE_ENV=production \
                        --expose ${APP_PORT} \
                        ${IMAGE_NAME}:latest
                    """
                }
            }
        }

        stage('4. Refresh Nginx') {
            steps {
                script {
                    sh "docker exec ${NGINX_CONTAINER} nginx -s reload"
                }
            }
        }
    }
    
    post {
        always {
            sh "docker image prune -f" 
        }
    }
}