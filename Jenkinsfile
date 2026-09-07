pipeline {
    agent any

    environment {
        REPORTS_DIR = 'security-reports'
        SONAR_HOST_URL = 'http://localhost:9000'
        SONAR_TOKEN = 'sqa_c18b9398b7904f6dce239a5d4902c0b39ef776d0'
        SONAR_PROJECT_KEY = 'Testing'
    }

    stages {
        stage('Install Dependencies') {
            steps {
                sh '''
                    # Install semgrep via pip if not present
                    if ! command -v semgrep &>/dev/null; then
                        pip3 install semgrep || pip3 install --user semgrep
                    fi

                    # Install python3 if not present (use apt on Debian/Ubuntu)
                    if ! command -v python3 &>/dev/null; then
                        sudo apt-get update && sudo apt-get install -y python3 python3-pip
                    fi

                    # Install docker if not present
                    if ! command -v docker &>/dev/null; then
                        sudo apt-get update && sudo apt-get install -y docker.io
                        sudo systemctl start docker
                    fi

                    # Install curl (used by sonarqube-scan.sh for polling)
                    if ! command -v curl &>/dev/null; then
                        sudo apt-get install -y curl
                    fi
                '''
            }
        }

        stage('Security Analysis') {
            steps {
                script {
                    sh 'mkdir -p $REPORTS_DIR'

                    // Run Semgrep and SonarQube scans in parallel
                    parallel (
                        'Semgrep SAST': {
                            sh 'chmod +x scripts/semgrep-scan.sh'
                            sh './scripts/semgrep-scan.sh . $REPORTS_DIR'
                        },
                        'SonarQube Analysis': {
                            sh 'chmod +x scripts/sonarqube-scan.sh'
                            sh './scripts/sonarqube-scan.sh . $SONAR_PROJECT_KEY $SONAR_HOST_URL $SONAR_TOKEN $REPORTS_DIR'
                        }
                    )
                }
            }
        }

        stage('Deduplicate Findings') {
            steps {
                sh 'python3 scripts/security_processor.py --workspace $REPORTS_DIR'
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution completed.'
            archiveArtifacts artifacts: "$REPORTS_DIR/*.json, $REPORTS_DIR/*.log", allowEmptyArchive: true
        }
    }
}
