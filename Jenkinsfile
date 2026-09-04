pipeline {
    agent any

    stages {
        stage('Security Analysis') {
            when {
                changeRequest()
            }
            parallel {
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
            echo 'Pipeline execution completed.'
            archiveArtifacts artifacts: 'semgrep-results.json', allowEmptyArchive: true
        }
    }
}
