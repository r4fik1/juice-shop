pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('github-token') // ID de tu credencial de GitHub
        SEMGREP_OUTPUT = "semgrep-results.json"
    }
    stages {
        stage('Clonar el Repositorio') {
            steps {
                node('master') { // Cambiado a 'master'
                    sh """
                    git clone https://$GITHUB_TOKEN@github.com/r4fik1/juice-shop.git
                    """
                }
            }
        }
        stage('Ejecutar Semgrep') {
            steps {
                node('master') { // Cambiado a 'master'
                    sh """
                    cd juice-shop
                    semgrep --config p/owasp-top-ten --json --output $SEMGREP_OUTPUT .
                    """
                }
            }
        }
        stage('Publicar Resultados') {
            steps {
                node('master') { // Cambiado a 'master'
                    sh "cat juice-shop/$SEMGREP_OUTPUT"
                }
            }
        }
    }
    post {
        always {
            node('master') { // Cambiado a 'master'
                archiveArtifacts artifacts: "juice-shop/$SEMGREP_OUTPUT", onlyIfSuccessful: true
            }
        }
    }
}
