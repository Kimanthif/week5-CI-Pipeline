pipeline {
    agent any


    environment {
        NEXUS_URL = "http://localhost:8081"
        NEXUS_REPO = "kijanikiosk-payments"
        NEXUS_CREDENTIALS_ID = "nexus-creds"
        PATH = "/usr/local/bin:/usr/bin:/bin:/usr/sbin:/sbin:${env.PATH}"

        GIT_SHA = "${env.GIT_COMMIT.take(7)}"
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
                sh 'npm run lint'
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
                        sh 'npm audit --audit-level=high'
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

                    sh """
                        echo "Preparing package version ${VERSION}"

                        npm version ${VERSION} --no-git-tag-version

                        cat > .npmrc <<EOF
registry=${NEXUS_URL}/repository/${NEXUS_REPO}/
_auth=\$(echo -n $NEXUS_USER:$NEXUS_PASS | base64)
EOF

                        npm publish --registry ${NEXUS_URL}/repository/${NEXUS_REPO}/

                        rm -f .npmrc
                    """
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