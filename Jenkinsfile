pipeline {
    agent any

    environment {
        APP_NAME = "netflix-clone"
        // Dynamically tagging the image with the build number (e.g., netflix:21, netflix:22)
        // This is the best way to ensure you never see "old code"
        IMAGE_NAME = "netflix:${env.BUILD_NUMBER}"
    }

    stages {
        stage('Clean Start') {
            steps {
                echo "Wiping workspace to ensure fresh code from GitHub..."
                cleanWs()
            }
        }

        stage('Checkout Code') {
            steps {
                echo "Fetching latest code from main branch..."
                git branch: 'main', url: 'https://github.com/kaybee-singh/netflix-clone/'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    echo "Building Image: ${IMAGE_NAME}"
                    // --no-cache is your best friend for live demos
                    sh "sudo docker build --no-cache -t ${IMAGE_NAME} -f Containerfile ."
                }
            }
        }

        stage('Deploy (Atomic Swap)') {
            steps {
                script {
                    echo "Killing old container and launching ${IMAGE_NAME}..."
                    // Force remove old container to clear the port
                    sh "sudo docker rm -f ${APP_NAME} || true"
                    
                    // Launch new container
                    sh "sudo docker run -d --name ${APP_NAME} -p 80:80 ${IMAGE_NAME}"
                }
            }
        }

        stage('Verification') {
            steps {
                script {
                    echo "Checking if the new code reached the container..."
                    // This command will print the title in your Jenkins logs
                    sh "sudo docker exec ${APP_NAME} grep -i 'Kubearc' /usr/share/nginx/html/index.html || echo 'Title not found! Check your Git push.'"
                }
            }
        }
    }

    post {
        success {
            echo "--------------------------------------------------------"
            echo "DEPLOYMENT SUCCESSFUL (Build #${env.BUILD_NUMBER})"
            echo "Refresh your browser (Ctrl + F5) to see the changes."
            echo "--------------------------------------------------------"
        }
        always {
            // Optional: Clean up old images to save disk space on EC2
            // sh "sudo docker image prune -f"
        }
    }
}
