pipeline {
    agent any
    
    // ============================================================
    // KHU VỰC CẤU HÌNH (CHỈ CẦN SỬA Ở ĐÂY)
    // ============================================================
    environment {
        // 1. CẤU HÌNH GIT
        GIT_REPO_URL   = 'https://github.com/cnguyenmanh26/vietmythluminarts-api.git'
        GIT_BRANCH     = 'main'

        // 2. CẤU HÌNH DOCKER CONTAINER
        // Tên container (Bắt buộc phải là 'backend' để khớp với Nginx của bạn)
        CONTAINER_NAME = 'backend'
        // Tên Image (Đặt tùy ý)
        IMAGE_NAME     = 'vietmyth-backend-image'
        
        // 3. CẤU HÌNH MẠNG & MÔI TRƯỜNG SERVER
        // Tên mạng Docker (Kiểm tra bằng lệnh: docker network ls)
        NETWORK_NAME   = 'home_default'
        // Đường dẫn tuyệt đối tới file .env trên Server
        ENV_FILE_PATH  = '/home/back-end/.env'
        // Port nội bộ của ứng dụng (NodeJS thường là 3000 hoặc 5000)
        APP_PORT       = '5000'
    }
    // ============================================================
    // HẾT PHẦN CẤU HÌNH - KHÔNG CẦN SỬA DƯỚI NÀY
    // ============================================================

    stages {
        stage('1. Checkout Code') {
            steps {
                script {
                    echo "--- Đang lấy code từ nhánh: ${GIT_BRANCH} ---"
                    git branch: "${GIT_BRANCH}", url: "${GIT_REPO_URL}"
                }
            }
        }

        stage('2. Build Docker Image') {
            steps {
                script {
                    echo "--- Đang Build Image: ${IMAGE_NAME} ---"
                    // Build từ thư mục gốc (.)
                    sh "docker build -t ${IMAGE_NAME}:latest ."
                }
            }
        }

        stage('3. Deploy Container') {
            steps {
                script {
                    echo "--- Đang Deploy Container: ${CONTAINER_NAME} ---"
                    
                    // 1. Dừng container cũ
                    sh "docker stop ${CONTAINER_NAME} || true"
                    
                    // 2. Xóa container cũ
                    sh "docker rm ${CONTAINER_NAME} || true"
                    
                    // 3. Chạy container mới với các biến đã khai báo
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
    }
    
    post {
        success {
            echo "✅ DEPLOY THÀNH CÔNG: ${CONTAINER_NAME}"
        }
        failure {
            echo "❌ DEPLOY THẤT BẠI: Vui lòng kiểm tra Log"
        }
        always {
            // Dọn dẹp image rác
            sh "docker image prune -f"
        }
    }
}