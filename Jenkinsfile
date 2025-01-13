pipeline {
    agent any
    environment {
        SEMGREP_HOME = "/tmp/.semgrep"
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
        failure {
            echo 'Pipeline failed. Check logs for details.'
        }
    }
}
