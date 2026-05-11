pipeline {
    agent any

    environment {
        NEXUS_URL = "http://172.17.0.1:8081"
        NEXUS_REPO = "kijanikiosk-payments"
        NEXUS_CREDENTIALS_ID = "nexus-creds"

        PATH = "/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:${env.PATH}"

        GIT_SHA = "${env.GIT_COMMIT?.take(7) ?: 'dev'}"
        BASE_VERSION = "1.0.0"
        VERSION = "${BASE_VERSION}-${GIT_SHA}"
    }

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    stages {

        stage('Debug Node') {
            steps {
                sh 'node -v && npm -v'
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }

        stage('Lint') {
            steps {
                sh 'npm run lint || echo "Lint not configured, continuing pipeline"'
            }
        }

        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }

        stage('Verify') {
            parallel {
                stage('Test') {
                    steps {
                        sh 'npm test -- --ci'
                    }
                }

                stage('Security Audit') {
                    steps {
                        sh 'npm audit --audit-level=high || true'
                    }
                }
            }
        }

        stage('Archive') {
            steps {
                archiveArtifacts artifacts: 'dist/**', fingerprint: true
            }
        }

        stage('Publish to Nexus') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: "${NEXUS_CREDENTIALS_ID}",
                    usernameVariable: 'NEXUS_USER',
                    passwordVariable: 'NEXUS_PASS'
                )]) {

                    sh '''
                        set -e

                        echo "Preparing package version ${VERSION}"

                        npm version ${VERSION} --no-git-tag-version

                        AUTH=$(echo -n "$NEXUS_USER:$NEXUS_PASS" | base64)

                        cat > .npmrc <<EOF
registry=${NEXUS_URL}/repository/${NEXUS_REPO}/
always-auth=true
//localhost:8081/repository/${NEXUS_REPO}/:_auth=${AUTH}
EOF

                        npm publish --registry ${NEXUS_URL}/repository/${NEXUS_REPO}/

                        rm -f .npmrc
                    '''
                }
            }
        }
    }

    post {
        always {
            echo "Cleaning workspace..."
            cleanWs()
        }

        success {
            echo "Build successful! Artifact published with version: ${VERSION}"
        }

        failure {
            echo "Pipeline failed. Check logs for details."
        }

        changed {
            echo "Pipeline status changed from previous run."
        }
    }
}