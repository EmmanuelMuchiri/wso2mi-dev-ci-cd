pipeline {
    agent any

    tools {
        maven 'MVN_HOME'
    }

    environment {
        MI_HOME = '/host_ci_cd/wso2mi-4.3.0'   // already extracted locally
        CAR_DEPLOY_DIR = "${MI_HOME}/repository/deployment/server/carbonapps"

        DOCKER_IMAGE_NAME = "emmanuelmuchiri/dbg"
        DOCKER_IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Check Environment') {
            steps {
                sh '''
                echo "===== Checking Build Environment ====="
                mvn -version
                java -version
                
                echo "MI_HOME = $MI_HOME"
                ls -lah $MI_HOME || exit 1
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
                echo "===== Building CAR ====="
                mvn clean install -DskipTests
                '''
            }
        }

        stage('Deploy CAR to MI Runtime') {
            steps {
                sh '''
                echo "===== Deploying CAR to MI Runtime ====="

                CAR_FILE=$(find . -name "*.car" | head -n 1)

                if [ -z "$CAR_FILE" ]; then
                    echo "❌ No CAR file found."
                    exit 1
                fi

                echo "Found CAR: $CAR_FILE"

                mkdir -p ${CAR_DEPLOY_DIR}

                echo "Removing old CARs..."
                rm -f ${CAR_DEPLOY_DIR}/*.car

                echo "Copying new CAR..."
                cp "$CAR_FILE" ${CAR_DEPLOY_DIR}/

                echo "CAR deployed to ${CAR_DEPLOY_DIR}"
                '''
            }
        }

        stage('Generate Dockerfile') {
            steps {
                writeFile file: 'Dockerfile', text: """
FROM eclipse-temurin:11-jre

ENV MI_HOME=/opt/wso2mi-4.3.0

COPY ${MI_HOME}/ \$MI_HOME/

WORKDIR \$MI_HOME

RUN chmod +x \$MI_HOME/bin/*.sh

EXPOSE 8290 8253 9164

ENTRYPOINT ["sh", "-c", "\$MI_HOME/bin/micro-integrator.sh"]
"""
                sh "echo 'Dockerfile created.'"
            }
        }

        stage('Build Docker Image') {
            steps {
                sh '''
                echo "===== Building Docker Image ====="
                docker build -t ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} .
                docker tag ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG} ${DOCKER_IMAGE_NAME}:latest
                '''
            }
        }

        stage('Push to DockerHub') {
            environment {
                DOCKER_CREDS = credentials('dockerhub-creds')
            }
            steps {
                sh '''
                echo "===== DockerHub Login ====="
                echo "$DOCKER_CREDS_PSW" | docker login -u "$DOCKER_CREDS_USR" --password-stdin

                echo "===== Pushing Images ====="
                docker push ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
                docker push ${DOCKER_IMAGE_NAME}:latest
                '''
            }
        }
    }

    post {
        success {
            echo """
            🎉 SUCCESS — CI/CD Pipeline Completed!

            Docker Images:
              - ${DOCKER_IMAGE_NAME}:${DOCKER_IMAGE_TAG}
              - ${DOCKER_IMAGE_NAME}:latest
            """
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
