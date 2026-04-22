pipeline {
    agent any

    environment {
        APP_NAME = "netflix-clone"
        IMAGE_NAME = "netflix:${env.BUILD_NUMBER}"
    }

    stages {
        stage('Cleanup') {
            steps {
                echo "Wiping old workspace..."
                cleanWs()
            }
        }

        stage('Checkout') {
            steps {
                echo "Fetching code from MASTER branch..."
                git branch: 'master', url: 'https://github.com/kaybee-singh/netflix1'
            }
        }

        stage('Docker Build') {
            steps {
                script {
                    echo "Building Image: ${IMAGE_NAME}..."
                    sh "sudo docker build --no-cache -t ${IMAGE_NAME} -f Containerfile ."
                }
            }
        }

        stage('Deploy') {
            steps {
                script {
                    echo "Removing old container and starting Build #${env.BUILD_NUMBER}..."
                    sh "sudo docker rm -f ${APP_NAME} || true"
                    sh "sudo docker run -d --name ${APP_NAME} -p 80:80 ${IMAGE_NAME}"
                }
            }
        }

        stage('Verify') {
            steps {
                script {
                    // This prints the line containing 'Kubearc' to the Jenkins console
                    sh "sudo docker exec ${APP_NAME} grep -i 'Kubearc' /usr/share/nginx/html/index.html || echo 'Update not found!'"
                }
            }
        }
    }

    post {
        success {
            echo "Successfully deployed Build #${env.BUILD_NUMBER}"
        }
        failure {
            echo "Build failed. Check the logs above."
        }
    }
}
