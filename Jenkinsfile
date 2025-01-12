pipeline {
    agent any
    stages {
        stage('Run Dependency Track (SCA)') {
            agent {
                docker {
                    image 'maven:3.8.5-openjdk-11'
                }
            }
            steps {
                script {
                    sh """
                    mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom
                    curl -X POST -H 'X-Api-Key: $DEPENDENCY_TRACK_API_KEY' \
                         -F "bom=@target/bom.xml" \
                         $DEPENDENCY_TRACK_URL
                    """
                }
            }
        }

        stage('Run SAST - Semgrep') {
            steps {
                script {
                    sh """
                    semgrep --config auto --json --output semgrep-results.json
                    curl -X POST -H "Authorization: Token $DEFECTDOJO_API_KEY" \
                         -H "Content-Type: application/json" \
                         -d @semgrep-results.json \
                         $DEFECTDOJO_URL
                    """
                }
            }
        }

        stage('Run DAST - Nuclei') {
            steps {
                script {
                    sh """
                    nuclei -u http://localhost:3000 -json -o nuclei-results.json
                    curl -X POST -H "Authorization: Token $DEFECTDOJO_API_KEY" \
                         -H "Content-Type: application/json" \
                         -d @nuclei-results.json \
                         $DEFECTDOJO_URL
                    """
                }
            }
        }

        stage('Run Container Security - Trivy') {
            steps {
                script {
                    sh """
                    trivy image --format json -o trivy-results.json bkimminich/juice-shop
                    curl -X POST -H "Authorization: Token $DEFECTDOJO_API_KEY" \
                         -H "Content-Type: application/json" \
                         -d @trivy-results.json \
                         $DEFECTDOJO_URL
                    """
                }
            }
        }

        stage('Run Secrets Scanning - Gitleaks') {
            steps {
                script {
                    sh """
                    gitleaks detect --source . --report-format json --report-path gitleaks-results.json
                    curl -X POST -H "Authorization: Token $DEFECTDOJO_API_KEY" \
                         -H "Content-Type: application/json" \
                         -d @gitleaks-results.json \
                         $DEFECTDOJO_URL
                    """
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline execution completed.'
        }
        failure {
            echo 'Pipeline failed. Please check logs for details.'
        }
    }
}
