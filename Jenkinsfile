pipeline {
    agent any
    environment {
        // URLs de integración y credenciales
        DEPENDENCY_TRACK_URL = 'http://172.17.0.2:8080/api/v1/bom'
        DEPENDENCY_TRACK_API_KEY = credentials('DEPENDENCY_TRACK_API_KEY')
        DEFECTDOJO_URL = 'http://localhost:8080/api/v2/import-scan/'
        DEFECTDOJO_API_KEY = credentials('DEFECTDOJO_API_KEY')
        // The following variable is required for a Semgrep AppSec Platform-connected scan:
        SEMGREP_APP_TOKEN = credentials('SEMGREP_APP_TOKEN')

        // Uncomment to scan changed files in PRs or MRs (diff-aware scanning):
        // SEMGREP_BASELINE_REF = "main"

        // Troubleshooting:

        // Uncomment the following lines if Semgrep AppSec Platform > Findings Page does not create links
        // to the code that generated a finding or if you are not receiving PR or MR comments.
        // SEMGREP_JOB_URL = "${BUILD_URL}"
        // SEMGREP_COMMIT = "${GIT_COMMIT}"
        // SEMGREP_BRANCH = "${GIT_BRANCH}"
        // SEMGREP_REPO_NAME = env.GIT_URL.replaceFirst(/^https:\/\/github.com\/(.*).git$/, '$1')
        // SEMGREP_REPO_URL = env.GIT_URL.replaceFirst(/^(.*).git$/,'$1')
        // SEMGREP_PR_ID = "${env.CHANGE_ID}"
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
    steps {
        script {
            // Asegúrate de tener el entorno virtual activado antes de ejecutar Semgrep
            sh '''
            python3 -m venv /root/semgrep-venv
            source /root/semgrep-venv/bin/activate
            pip install semgrep
            semgrep --config p/ci --metrics=off --json --output semgrep-results.json --debug
            if [ -s semgrep-results.json ]; then
                curl -X POST -H "Authorization: Token $DD_API_KEY" \
                     -H "Content-Type: application/json" \
                     -d @semgrep-results.json \
                     $DEFECTDOJO_URL
            else
                echo "Semgrep results are empty. Skipping upload to DefectDojo."
            fi
            deactivate
            '''
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
