pipeline {
    agent any
    environment {
        GITHUB_TOKEN = credentials('github-token') // ID de tu credencial
    }
    stages {
        stage('Clonar el Repositorio') {
            steps {
                // Clonar el repositorio usando el token
                sh """
                git clone https://$GITHUB_TOKEN@github.com/r4fik1/juice-shop.git
                """
            }
        }
        stage('Ejecutar Semgrep') {
            steps {
                // Ejecutar análisis con Semgrep
                sh """
                cd juice-shop
                semgrep --config p/owasp-top-ten --json --output semgrep-results.json .
                """
            }
        }
        stage('Publicar Resultados') {
            steps {
                // Mostrar los resultados en la consola
                sh "cat juice-shop/semgrep-results.json"
            }
        }
    }
    post {
        always {
            // Archivar los resultados como artefacto
            archiveArtifacts artifacts: 'juice-shop/semgrep-results.json', onlyIfSuccessful: true
        }
    }
}
