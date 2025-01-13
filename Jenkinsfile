pipeline {
    agent any
    environment {
        SEMGREP_HOME = "${WORKSPACE}/.semgrep" // Redirige Semgrep a un directorio en el workspace
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
                    # Crea el directorio SEMGREP_HOME
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
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
