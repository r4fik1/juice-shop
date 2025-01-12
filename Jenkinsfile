pipeline {
    agent any
    environment {
        // URLs de integración y credenciales
        DEPENDENCY_TRACK_URL = 'http://172.17.0.2:8080/api/v1/bom'
        DEPENDENCY_TRACK_API_KEY = credentials('DEPENDENCY_TRACK_API_KEY')
        DEFECTDOJO_URL = 'http://localhost:8080/api/v2/import-scan/'
        DEFECTDOJO_API_KEY = credentials('DEFECTDOJO_API_KEY')
    }
    stages {
        stage('Run Dependency Track (SCA)') {
            agent {
                docker {
                    image 'maven:3.8.5-openjdk-11'
                }
            }
            steps {
                script {
                    sh '''
                    mvn org.cyclonedx:cyclonedx-maven-plugin:makeAggregateBom
                    if [ -f target/bom.xml ]; then
                        curl -X POST -H "X-Api-Key: $DEPENDENCY_TRACK_API_KEY" \
                             -F "bom=@target/bom.xml" \
                             $DEPENDENCY_TRACK_URL
                    else
                        echo "SBOM not found. Skipping upload to Dependency Track."
                    fi
                    '''
                }
            }
        }

        stage('Run SAST - Semgrep') {
            agent {
                docker {
                    image 'returntocorp/semgrep:latest'
                }
            }
            steps {
                withCredentials([string(credentialsId: 'DEFECTDOJO_API_KEY', variable: 'DD_API_KEY')]) {
                    script {
                        sh '''
                        # Ejecuta Semgrep y genera el archivo de resultados
                        semgrep --config p/ci --metrics=off --json --output semgrep-results.json --debug
                        # Verifica si el archivo tiene contenido y realiza la subida a DefectDojo
                        if [ -s semgrep-results.json ]; then
                            curl -X POST -H "Authorization: Token $DD_API_KEY" \
                                -H "Content-Type: application/json" \
                                -d @semgrep-results.json \
                                $DEFECTDOJO_URL
                        else
                            echo "Semgrep results are empty. Skipping upload to DefectDojo."
                        fi
                        '''
                    }
                }
            }
        }

        stage('Run Container Security - Trivy') {
            agent {
                docker {
                    image 'aquasec/trivy:latest'
                }
            }
            steps {
                withCredentials([string(credentialsId: 'DEFECTDOJO_API_KEY', variable: 'DD_API_KEY')]) {
                    script {
                        sh '''
                        trivy image --format json -o trivy-results.json bkimminich/juice-shop
                        if [ -s trivy-results.json ]; then
                            curl -X POST -H "Authorization: Token $DD_API_KEY" \
                                 -H "Content-Type: application/json" \
                                 -d @trivy-results.json \
                                 $DEFECTDOJO_URL
                        else
                            echo "Trivy results are empty. Skipping upload to DefectDojo."
                        fi
                        '''
                    }
                }
            }
        }

        stage('Run Secrets Scanning - Gitleaks') {
            agent {
                docker {
                    image 'zricethezav/gitleaks:latest'
                }
            }
            steps {
                withCredentials([string(credentialsId: 'DEFECTDOJO_API_KEY', variable: 'DD_API_KEY')]) {
                    script {
                        sh '''
                        gitleaks detect --source . --report-format json --report-path gitleaks-results.json
                        if [ -s gitleaks-results.json ]; then
                            curl -X POST -H "Authorization: Token $DD_API_KEY" \
                                 -H "Content-Type: application/json" \
                                 -d @gitleaks-results.json \
                                 $DEFECTDOJO_URL
                        else
                            echo "Gitleaks results are empty. Skipping upload to DefectDojo."
                        fi
                        '''
                    }
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
