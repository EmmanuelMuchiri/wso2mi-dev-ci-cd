pipeline {
    agent any

    tools {
        maven 'MVN_HOME' // Ensure this matches the Maven tool name configured in Jenkins
    }

    environment {
        // ✅ Mounted path of Micro Integrator inside the Jenkins container
        MI_HOME = '/opt/wso2mi'
        MI_SCRIPT = "${MI_HOME}/bin/micro-integrator.sh"
        CAR_DEPLOY_DIR = "${MI_HOME}/repository/deployment/server/carbonapps"

        // Add system binaries to PATH if needed
        PATH = "/usr/local/bin:/usr/bin:/bin:${env.PATH}"
    }

    stages {

        stage('Check Environment') {
            steps {
                sh '''
                echo "===== Checking Environment ====="
                which mvn || echo "Maven not found!"
                which java || echo "Java not found!"
                which zip || echo "zip not found!"
                java -version
                echo "MI_HOME = $MI_HOME"
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
                if [ -f "$MI_SCRIPT" ]; then
                    sh $MI_SCRIPT stop
                else
                    echo "Micro Integrator script not found at $MI_SCRIPT"
                    exit 1
                fi

                sleep 5

                # Confirm shutdown
                if pgrep -f micro-integrator > /dev/null; then
                    echo "Warning: Micro Integrator still running!"
                    ps aux | grep micro-integrator | grep -v grep
                else
                    echo "✅ Micro Integrator stopped successfully."
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
                if [ -f "$MI_SCRIPT" ]; then
                    nohup sh $MI_SCRIPT start &>/dev/null &
                else
                    echo "Micro Integrator script not found at $MI_SCRIPT"
                    exit 1
                fi

                sleep 10

                # Confirm startup
                if pgrep -f micro-integrator > /dev/null; then
                    echo "✅ Micro Integrator started successfully."
                else
                    echo "❌ Failed to start Micro Integrator!"
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
            echo "❌ Deployment failed. Please check the logs and fix the issue."
        }
    }
}
