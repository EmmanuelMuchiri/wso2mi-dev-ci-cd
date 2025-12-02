pipeline {
    agent any

    tools {
        maven 'MVN_HOME'
    }

    environment {
        MI_HOME = '/host_ci_cd/wso2mi-4.3.0'
        CAR_DEPLOY_DIR = "${MI_HOME}/repository/deployment/server/carbonapps"
        DOCKER_IMAGE_NAME = 'wso2mi-custom'
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
        PATH = "/usr/local/bin:/usr/bin:/bin:${env.PATH}"
    }

    stages {

        stage('Check Environment') {
            steps {
                sh '''
                echo "===== Checking Environment ====="
                which mvn || echo "Maven not found!"
                which java || echo "Java not found!"
                java -version
                echo "MI_HOME = $MI_HOME"
                echo "CAR_DEPLOY_DIR = $CAR_DEPLOY_DIR"
                
                if [ -d "$MI_HOME" ]; then
                    echo "✅ MI_HOME directory accessible"
                    ls -la $MI_HOME
                else
                    echo "❌ MI_HOME directory not found!"
                    exit 1
                fi
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

        stage('Create Dockerfile') {
            steps {
                sh '''
                echo "===== Creating Dockerfile ====="
                
                cat > /host_ci_cd/Dockerfile <<'EOF'
FROM ubuntu:20.04

ENV DEBIAN_FRONTEND=noninteractive
ENV JAVA_HOME=/usr/lib/jvm/java-11-openjdk-amd64
ENV MI_HOME=/opt/wso2mi-4.3.0

RUN apt-get update && \
    apt-get install -y openjdk-11-jdk && \
    apt-get clean && \
    rm -rf /var/lib/apt/lists/*

COPY wso2mi-4.3.0 ${MI_HOME}

WORKDIR ${MI_HOME}

RUN chmod +x ${MI_HOME}/bin/*.sh

EXPOSE 8290 8253 9164

CMD ["sh", "-c", "${MI_HOME}/bin/micro-integrator.sh"]
EOF

                echo "✅ Dockerfile created at /host_ci_cd/Dockerfile"
                cat /host_ci_cd/Dockerfile
                '''
            }
        }

        stage('Build Docker Image') {
            steps {
                script {
                    // Create a build script that will run on the host
                    sh '''
                    echo "===== Creating Docker build script for host ====="
                    
                    cat > /host_ci_cd/build-docker.sh <<EOF
#!/bin/bash
set -e

echo "Building Docker image on host machine..."
cd /Users/emmanuelmuchiri/Documents/Kulana/CBG/CI_CD

docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .
docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_IMAGE_NAME}:latest

echo "✅ Docker image built successfully"
docker images | grep ${DOCKER_IMAGE_NAME}
EOF

                    chmod +x /host_ci_cd/build-docker.sh
                    
                    echo "✅ Build script created at /host_ci_cd/build-docker.sh"
                    echo ""
                    echo "Run this command on your Mac terminal:"
                    echo "/Users/emmanuelmuchiri/Documents/Kulana/CBG/CI_CD/build-docker.sh"
                    '''
                }
            }
        }

        stage('Manual Build Instructions') {
            steps {
                sh '''
                echo "=============================================="
                echo "DOCKER BUILD INSTRUCTIONS"
                echo "=============================================="
                echo ""
                echo "Since Docker CLI is not available in Jenkins,"
                echo "please run this command on your Mac:"
                echo ""
                echo "  /Users/emmanuelmuchiri/Documents/Kulana/CBG/CI_CD/build-docker.sh"
                echo ""
                echo "Or manually:"
                echo "  cd /Users/emmanuelmuchiri/Documents/Kulana/CBG/CI_CD"
                echo "  docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ."
                echo "  docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_IMAGE_NAME}:latest"
                echo ""
                echo "=============================================="
                '''
            }
        }
    }

    post {
        success {
            echo """
            ✅ Pipeline completed successfully!
            
            Next steps:
            1. Run the build script on your Mac:
               /Users/emmanuelmuchiri/Documents/Kulana/CBG/CI_CD/build-docker.sh
            
            2. Then start the container:
               docker run -d -p 8290:8290 -p 8253:8253 -p 9164:9164 --name wso2mi ${DOCKER_IMAGE_NAME}:latest
            
            3. View logs:
               docker logs -f wso2mi
            """
        }
        failure {
            echo "❌ Pipeline failed. Please check the logs."
        }
    }
}
