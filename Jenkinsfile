pipeline {
    agent any

    environment {
        DOCKER_COMPOSE_FILE = 'docker-compose.yml'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Start Docker Compose') {
            steps {
                sh '''
                    echo "[*] Konteynerlər qaldırılır..."
                    docker-compose -f ${DOCKER_COMPOSE_FILE} up -d
                '''
            }
        }

        stage('Install Python & Semgrep') {
            steps {
                sh '''
                    echo "[*] Python, venv və Semgrep quraşdırılır..."

                    apt-get update -y
                    apt-get install -y python3 python3-venv

                    python3 -m venv venv
                    . venv/bin/activate

                    pip install --upgrade pip
                    pip install semgrep
                '''
            }
        }

        stage('Run Semgrep Scan') {
            steps {
                sh '''
                    echo "[*] Semgrep scan başlayır..."

                    . venv/bin/activate
                    semgrep --config=auto --output=semgrep-results.json --json .
                '''
            }
        }

        stage('Stop containers') {
            steps {
                sh '''
                    echo "[*] Konteynerlər dayandırılır..."
                    docker-compose -f ${DOCKER_COMPOSE_FILE} down
                '''
            }
        }
    }

    post {
        always {
            echo "[*] Scan nəticələri saxlanılır..."
            archiveArtifacts artifacts: 'semgrep-results.json', allowEmptyArchive: true
        }
    }
}
