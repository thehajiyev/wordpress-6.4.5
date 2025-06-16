pipeline {
    agent any

    stages {
        stage('Start containers') {
            steps {
                echo '[*] WordPress və MySQL konteynerləri qaldırılır...'
                sh 'docker compose up -d'
            }
        }

        stage('Run Semgrep Scan') {
            steps {
                echo '[*] Semgrep scan başlayır...'
                sh 'semgrep --config=auto --output=semgrep-results.json --json .'
            }
        }

        stage('Stop containers') {
            steps {
                echo '[*] Konteynerlər dayandırılır...'
                sh 'docker compose down'
            }
        }
    }
}
