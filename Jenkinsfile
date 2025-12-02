pipeline {
    agent any

    tools {
        maven 'MVN_HOME'
    }

    environment {
        // Local WSO2 MI directory on your Mac
        MI_HOME = '/Users/emmanuelmuchiri/Documents/Kulana/CBG/CI_CD/wso2mi-4.3.0'
        CAR_DEPLOY_DIR = "${MI_HOME}/repository/deployment/server/carbonapps"
        
        // Docker image configuration
        DOCKER_IMAGE_NAME = 'wso2mi-custom'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}" // or use 'latest'
        
        PATH = "/usr/local/bin:/usr/bin:/bin:${env.PATH}"
    }

    stages {

        stage('Check Environment') {
            steps {
                sh '''
                echo "===== Checking Environment ====="
                which mvn || echo "Maven not found!"
                which java || echo "Java not found!"
                which docker || echo "Docker not found!"
                java -version
                docker --version
                echo "MI_HOME = $MI_HOME"
                echo "CAR_DEPLOY_DIR = $CAR_DEPLOY_DIR"
                '''
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/EmmanuelMuchiri/wso2mi-dev-ci-cd.git'
            }
        }

        stage('Build CAR File') {
            steps {
                sh '''
                echo "===== Building CAR file ====="
                mvn clean install
                '''
            }
        }

        stage('Deploy CAR to Local MI') {
            steps {
                sh '''
                echo "===== Deploying CAR file to local MI directory ====="
                
                CAR_FILE=$(find . -name "*.car" | head -n 1)
                echo "CAR File to deploy: $CAR_FILE"

                if [ -f "$CAR_FILE" ]; then
                    echo "Creating deployment directory if it doesn't exist..."
                    mkdir -p ${CAR_DEPLOY_DIR}
                    
                    echo "Removing old CAR files..."
                    rm -f ${CAR_DEPLOY_DIR}/*.car

                    echo "Copying new CAR file to deployment directory..."
                    cp "$CAR_FILE" ${CAR_DEPLOY_DIR}/
                    
                    echo "✅ CAR file deployed to: ${CAR_DEPLOY_DIR}"
                    ls -lh ${CAR_DEPLOY_DIR}/
                else
                    echo "❌ CAR file not found! Failing pipeline."
                    exit 1
                fi
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "===== Building Docker Image ====="
                
                # Create Dockerfile if it doesn't exist
                cat > Dockerfile <<'EOF'
FROM ubuntu:20.04

# Set environment variables
ENV DEBIAN_FRONTEND=noninteractive
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
ENV MI_HOME=/opt/wso2mi-4.3.0

# Install Java 11
RUN apt-get update && \
    apt-get install -y openjdk-11-jdk && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

# Copy the entire WSO2 MI directory
COPY wso2mi-4.3.0 ${MI_HOME}

# Set working directory
WORKDIR ${MI_HOME}

# Make scripts executable
RUN chmod +x ${MI_HOME}/bin/*.sh

# Expose default MI ports
EXPOSE 8290 8253 9164

# Start Micro Integrator
CMD ["sh", "-c", "${MI_HOME}/bin/micro-integrator.sh"]
EOF

                echo "Dockerfile created. Building Docker image..."
                
                # Build Docker image with the MI directory as context
                cd /Users/emmanuelmuchiri/Documents/Kulana/CBG/CI_CD
                docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .
                docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_IMAGE_NAME}:latest
                
                echo "✅ Docker image built successfully"
                docker images | grep ${DOCKER_IMAGE_NAME}
                '''
            }
        }

        stage('Verify Docker Image') {
            steps {
                sh '''
                echo "===== Verifying Docker Image ====="
                docker images ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                
                echo "Image size and details:"
                docker inspect ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} | grep -A 5 "Size"
                '''
            }
        }
    }

    post {
        success {
            echo """
            ✅ Pipeline completed successfully!
            
            Docker Image: ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
            CAR File deployed to: ${CAR_DEPLOY_DIR}
            
            To run the container:
            docker run -d -p 8290:8290 -p 8253:8253 -p 9164:9164 --name wso2mi ${DOCKER_IMAGE_NAME}:latest
            """
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs."
        }
        always {
            sh '''
            echo "===== Cleaning up build artifacts ====="
            # Optional: clean workspace or temporary files
            '''
        }
    }
}
