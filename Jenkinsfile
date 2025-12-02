pipeline {
    agent any

    tools {
        maven 'MVN_HOME'
    }

    environment {
        MI_HOME = 'wso2mi-4.3.0'
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
                '''
            }
        }

        stage('Clone Repository') {
            steps {
                git branch: 'master', url: 'https://github.com/EmmanuelMuchiri/wso2mi-dev-ci-c-d.git'
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

        stage('Assemble Deployment Folder') {
            steps {
                sh '''
                echo "===== Preparing MI Deployment Folder ====="

                # Create build workspace
                rm -rf build
                mkdir -p build/${MI_HOME}/repository/deployment/server/carbonapps

                # Extract runtime (assumes wso2mi-4.3.0.zip exists in repo)
                unzip -q ${MI_HOME}.zip -d build/

                # Copy CAR
                CAR_FILE=$(find . -name "*.car" | head -n 1)
                echo "CAR Found: $CAR_FILE"
                cp "$CAR_FILE" build/${MI_HOME}/repository/deployment/server/carbonapps/

                echo "Deployment folder ready."
                '''
            }
        }

        stage('Create Dockerfile') {
            steps {
                writeFile file: 'Dockerfile', text: """
FROM eclipse-temurin:11-jre

ENV MI_HOME=/opt/wso2mi-4.3.0

COPY build/wso2mi-4.3.0/ \$MI_HOME/

WORKDIR \$MI_HOME

RUN chmod +x \$MI_HOME/bin/*.sh

EXPOSE 8290 8253 9164

ENTRYPOINT ["sh", "-c", "\$MI_HOME/bin/micro-integrator.sh"]
"""
                sh "echo '===== Dockerfile ready ====='"
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
                DOCKER_CREDS = credentials('dockerhub-credentials')
            }
            steps {
                sh '''
                echo "===== Authenticating to DockerHub ====="
                echo "$DOCKER_CREDS_PSW" | docker login -u "$DOCKER_CREDS_USR" --password-stdin

                echo "===== Pushing Docker Images ====="
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

            Run MI Anywhere:
              docker run -d -p 8290:8290 -p 8253:8253 -p 9164:9164 --name mi ${DOCKER_IMAGE_NAME}:latest

            Logs:
              docker logs -f mi
            """
        }

        failure {
            echo "❌ Pipeline failed. Check logs above."
        }
    }
}
