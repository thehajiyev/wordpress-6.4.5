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
                    docker compose -f ${DOCKER_COMPOSE_FILE} up -d
                '''
            }
        }

        stage('Install Python and Semgrep') {
            steps {
                sh '''
                    echo "[*] Python və pip quraşdırılır..."
                    sudo apt-get update
                    sudo apt-get install -y python3 python3-pip

                    echo "[*] Semgrep quraşdırılır..."
                    pip3 install semgrep
                '''
            }
        }

        stage('Run Semgrep Scan') {
            steps {
                sh '''
                    echo "[*] Semgrep scan başlayır..."
                    semgrep --config=auto --output=semgrep-results.json --json .
                '''
            }
        }

        stage('Stop containers') {
            steps {
                sh '''
                    echo "[*] Konteynerlər dayandırılır..."
                    docker compose -f ${DOCKER_COMPOSE_FILE} down
                '''
            }
        }
    }

    post {
        always {
            echo "[*] Nəticələr saxlanılır..."
            archiveArtifacts artifacts: 'semgrep-results.json', allowEmptyArchive: true
        }
    }
}
