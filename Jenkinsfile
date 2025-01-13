pipeline {
    agent any
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
                    # Ejecutar análisis de Semgrep con todas las reglas de seguridad disponibles
                    semgrep --config "p/security-audit" --json --output semgrep-results.json

                    # Mostrar los resultados en consola
                    if [ -s semgrep-results.json ]; then
                        echo "Semgrep analysis completed successfully. Results saved in semgrep-results.json."
                    else
                        echo "Semgrep did not find any issues."
                    fi
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
            echo 'Semgrep analysis completed without errors.'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
