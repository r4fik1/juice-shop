pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('github-token') // ID de tu credencial de GitHub
        SEMGREP_OUTPUT = "semgrep-results.json"
    }
    stages {
        stage('Clonar el Repositorio') {
            steps {
                // Ejecutar dentro de un nodo
                node {
                    sh """
                    git clone https://$GITHUB_TOKEN@github.com/r4fik1/juice-shop.git
                    """
                }
            }
        }
        stage('Ejecutar Semgrep') {
            steps {
                node {
                    // Ejecutar análisis con Semgrep
                    sh """
                    cd juice-shop
                    semgrep --config p/owasp-top-ten --json --output $SEMGREP_OUTPUT .
                    """
                }
            }
        }
        stage('Publicar Resultados') {
            steps {
                node {
                    // Mostrar los resultados en consola
                    sh "cat juice-shop/$SEMGREP_OUTPUT"
                }
            }
        }
    }
    post {
        always {
            node {
                // Archivar los resultados como artefactos
                archiveArtifacts artifacts: "juice-shop/$SEMGREP_OUTPUT", onlyIfSuccessful: true
            }
        }
    }
}
