pipeline {
    agent any

    environment {
        SONAR_SCANNER_HOME = tool 'SonarQubeScanner'
    }

    stages {
        stage('SAST') {
            // Correct placement for 'when'
    stages {
        stage('Security Analysis') {
            when {
                changeRequest()
            }
            parallel {
                stage('Semgrep') {
                    steps {
                        script {
                            echo "Starting Semgrep scan on PR #${env.CHANGE_ID}..."
                            sh 'semgrep scan --config=auto --json -o semgrep-results.json || true'
                        }
                    }
                    post {
                        always {
                            archiveArtifacts artifacts: 'semgrep-results.json', allowEmptyArchive: true
                        }
                    }
                }

                stage('SonarQube') {
                    steps {
                        script {
                            echo "Starting SonarQube analysis on PR #${env.CHANGE_ID}..."
                            withSonarQubeEnv('SonarQubeServer') {
                                sh """
                                    ${SONAR_SCANNER_HOME}/bin/sonar-scanner \
                                      -Dsonar.projectKey=${env.JOB_BASE_NAME} \
                                      -Dsonar.sources=. \
                                      -Dsonar.pullrequest.key=${env.CHANGE_ID} \
                                      -Dsonar.pullrequest.branch=${env.CHANGE_BRANCH} \
                                      -Dsonar.pullrequest.base=${env.CHANGE_TARGET}
                                """
                            }
                        }
                    }
                }
            }
        }

        stage('SonarQube Quality Gate') {
            // Also add 'when' here so Quality Gate only runs on PRs
            when {
                changeRequest()
            }
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
                stage('Semgrep SAST') {
                    steps {
                        echo 'Running Semgrep SAST scan...'
                        sh 'semgrep scan --config auto --json -o semgrep-results.json || true'
                    }
                }
                stage('SonarQube Analysis') {
                    steps {
                        echo 'Running SonarQube Code Analysis...'
                        // Replace with your SonarScanner CLI or scanner tool invocation
                        sh 'sonar-scanner -Dsonar.projectKey=Testing -Dsonar.sources=. || true'
                    }
                }
            }
        }
    }

    post {
        always {
            cleanWs()
            echo 'Pipeline execution completed.'
            archiveArtifacts artifacts: 'semgrep-results.json', allowEmptyArchive: true
        }
    }
}
