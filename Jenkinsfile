pipeline {
    agent any

    tools {
        maven 'MVN_HOME' // Match the name configured in Jenkins
    }

    environment {
        // Paths and credentials
        MI_HOME = '/opt/wso2mi-4.2.0'                // Path to Micro Integrator installation
        CAR_DEPLOY_DIR = "${MI_HOME}/repository/deployment/server/carbonapps"
        MI_START_SCRIPT = "${MI_HOME}/bin/micro-integrator.sh"
        MI_STOP_SCRIPT = "${MI_HOME}/bin/micro-integrator.sh"

        // Ensure required tools are in PATH
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
                git branch: 'wso2_mi_test', url: 'https://github.com/EmmanuelMuchiri/dbg.git'
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
                pkill -f micro-integrator || echo "Micro Integrator not running"
                sleep 5
                '''
            }
        }

        stage('Deploy CAR File') {
            steps {
                sh '''
                echo "===== Deploying CAR file ====="
                
                # Find the newly built .car file
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
                nohup ${MI_START_SCRIPT} &>/dev/null &
                sleep 10
                echo "Micro Integrator started."
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
