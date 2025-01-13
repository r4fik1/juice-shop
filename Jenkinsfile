pipeline {
    agent any
    stages {
        stage("Run SAST - Semgrep") {
            steps {
                catchError(buildResult: 'SUCCESS', stageResult: 'UNSTABLE') {
                    sh '''
                    docker run --rm -v ${PWD}:/src returntocorp/semgrep semgrep --config auto --output semgrep-output.json --json
                    '''
                }
            }
            post {
                always {
                    archiveArtifacts artifacts: 'semgrep-output.json', fingerprint: true
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
