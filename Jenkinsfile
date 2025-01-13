pipeline {
    agent any
    environment {
        SEMGREP_HOME = "${env.WORKSPACE}/.semgrep" // Directorio seguro en el workspace
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
                    mkdir -p $SEMGREP_HOME
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
            echo 'Semgrep analysis completed without errors.'
        }
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
