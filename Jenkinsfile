
pipeline {
    agent any

    environment {
        COMPOSE_DIR = '/home/ubuntu/gaariyaari-part2'
    }

    triggers {
        githubPush()
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
                echo "Workspace: ${env.WORKSPACE}"
                echo "Commit: ${env.GIT_COMMIT}"
            }
        }

        stage('Deploy Part II') {
            steps {
                withCredentials([string(
                    credentialsId: 'gaariyaari-jwt-secret',
                    variable: 'JWT_SECRET'
                )]) {
                    sh '''
                        set -eu
                        cd "$COMPOSE_DIR"

                        export APP_SOURCE="$WORKSPACE"

                        docker compose down --remove-orphans
                        docker compose up -d

                        docker compose ps
                    '''
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh '''
                    set -eu

                    echo "Waiting for frontend..."
                    frontend_ready=0

                    for i in $(seq 1 30); do
                        if curl -fsS http://localhost:8082/html/index.html >/dev/null; then
                            frontend_ready=1
                            break
                        fi
                        sleep 2
                    done

                    if [ "$frontend_ready" -ne 1 ]; then
                        echo "Frontend did not become ready."
                        exit 1
                    fi

                    echo "Frontend is responding."

                    echo "Waiting for backend..."
                    backend_ready=0

                    for i in $(seq 1 30); do
                        if curl -fsS http://localhost:5003/ >/dev/null; then
                            backend_ready=1
                            break
                        fi
                        sleep 2
                    done

                    if [ "$backend_ready" -ne 1 ]; then
                        echo "Backend did not become ready."
                        exit 1
                    fi

                    echo "Backend is responding."

                    echo "Checking containers..."
                    docker ps --filter "name=gaariyaari-"

                    echo "Part II deployment verified successfully."
                '''
            }
        }
    }

    post {
        success {
            script {
                def recipient = sh(
                    script: "git log -1 --format='%ae'",
                    returnStdout: true
                ).trim()

                if (recipient) {
                    emailext(
                        to: recipient,
                        subject: "SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: """Build succeeded.

Job: ${env.JOB_NAME}
Build: ${env.BUILD_NUMBER}
Branch: ${env.BRANCH_NAME}
Commit: ${env.GIT_COMMIT}

Part II application: http://13.60.111.25:8082
"""
                    )
                }
            }
        }

        failure {
            script {
                def recipient = sh(
                    script: "git log -1 --format='%ae' || true",
                    returnStdout: true
                ).trim()

                if (recipient) {
                    emailext(
                        to: recipient,
                        subject: "FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}",
                        body: "Build failed. Check Jenkins console output: ${env.BUILD_URL}"
                    )
                }
            }
        }
    }
}
