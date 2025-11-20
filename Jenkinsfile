pipeline {
    agent any
    
    // 1. KHẮC PHỤC LỖI DOCKER API (Client version too new)
    environment {
        DOCKER_API_VERSION = '1.43' 
        
        // ============================================================
        // KHU VỰC CẤU HÌNH
        // ============================================================
        // 1. CẤU HÌNH GIT
        GIT_REPO_URL    = 'https://github.com/cnguyenmanh26/vietmythluminarts-api.git'
        GIT_BRANCH      = 'main'

        // 2. CẤU HÌNH DOCKER CONTAINER
        CONTAINER_NAME  = 'backend'             // Bắt buộc khớp với Nginx
        IMAGE_NAME      = 'vietmyth-backend-image'

        // 3. CẤU HÌNH MẠNG & MÔI TRƯỜNG SERVER
        NETWORK_NAME    = 'home_default'        // Check lại: docker network ls
        
        // ĐƯỜNG DẪN FILE .ENV (Nằm trên Server thật - Cần đường dẫn tuyệt đối)
        // Đã sửa lỗi chính tả 'bac-end' -> 'back-end'
        ENV_FILE_PATH   = '/home/back-end/.env' 
        
        APP_PORT        = '5000'
    }

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
                    
                    // SỬA LỖI QUAN TRỌNG TẠI ĐÂY:
                    // Không dùng đường dẫn /home/... vì Jenkins build từ code Git vừa tải về.
                    // Dấu chấm (.) nghĩa là tìm Dockerfile ngay tại thư mục gốc của Git.
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