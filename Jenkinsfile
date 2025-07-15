pipeline {
    agent any

    tools {
        maven 'MVN_HOME' // Match the name configured in Jenkins
    }

    environment {
        // ✅ Correct MI installation path on your machine
        MI_HOME = '/opt/wso2mi'
        CAR_DEPLOY_DIR = "${MI_HOME}/repository/deployment/server/carbonapps"
        MI_START_SCRIPT = "${MI_HOME}/bin/sh micro-integrator.sh"
        MI_STOP_SCRIPT = "${MI_HOME}/bin/sh micro-integrator.sh"

        // Ensure tools like mvn, java, etc. are in PATH
        PATH = "/usr/local/bin:/usr/bin:/bin:${env.PATH}"
    }

    stages {

        stage('Check Environment') {
            steps {
                sh '''
                echo "===== Checking tools ====="
                which mvn || echo "Maven not found!"
                which zip || echo "zip not found!"
                echo "JAVA_HOME=$JAVA_HOME"
                java -version
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

        stage('Stop Micro Integrator') {
            steps {
                sh '''
                echo "===== Stopping Micro Integrator ====="
                ${MI_STOP_SCRIPT} stop
                sleep 5

                # Optional: Confirm it's stopped
                if pgrep -f micro-integrator > /dev/null; then
                  echo "Warning: Micro Integrator still running!"
                  ps aux | grep micro-integrator | grep -v grep
                else
                  echo "Micro Integrator stopped successfully."
                fi
                '''
            }
        }

        stage('Deploy CAR File') {
            steps {
                sh '''
                echo "===== Deploying CAR file ====="
                
                CAR_FILE=$(find . -name "*.car" | head -n 1)
                echo "CAR File to deploy: $CAR_FILE"

                if [ -f "$CAR_FILE" ]; then
                    echo "Removing old CAR files..."
                    rm -f ${CAR_DEPLOY_DIR}/*.car

                    echo "Copying new CAR file to deployment directory..."
                    cp "$CAR_FILE" ${CAR_DEPLOY_DIR}/
                else
                    echo "CAR file not found! Failing pipeline."
                    exit 1
                fi
                '''
            }
        }

        stage('Start Micro Integrator') {
            steps {
                sh '''
                echo "===== Starting Micro Integrator ====="
                nohup ${MI_START_SCRIPT} start &>/dev/null &
                sleep 10

                # Confirm MI is running
                if pgrep -f micro-integrator > /dev/null; then
                    echo "Micro Integrator started successfully."
                else
                    echo "Failed to start Micro Integrator!"
                    exit 1
                fi
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Micro Integrator deployment completed successfully."
        }
        failure {
            echo "❌ Deployment failed. Please check logs for details."
        }
    }
}
