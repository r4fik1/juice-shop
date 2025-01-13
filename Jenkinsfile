pipeline {
    agent any
    environment {
        // Define un directorio accesible para Semgrep
        SEMGREP_HOME = "${WORKSPACE}/.semgrep"
    }
    stages {
        stage('Run SAST - Semgrep') {
            agent {
                docker {
                    image 'returntocorp/semgrep:latest'
                }
            }
            steps {
                script {
                    sh '''
                    # Crea el directorio especificado en SEMGREP_HOME
                    mkdir -p $SEMGREP_HOME
                    
                    # Ejecuta Semgrep usando las reglas de seguridad audit
                    semgrep --config "p/security-audit" --json --output semgrep-results.json
                    '''
                }
            }
        }
    }
    post {
        always {
            echo 'Pipeline execution completed.'
        }
        success {
            echo 'Semgrep analysis completed successfully.'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
