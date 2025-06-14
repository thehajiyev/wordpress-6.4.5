pipeline {
    agent any

    environment {
        DOCKER_COMPOSE = "docker-compose"
    }

    stages {
        stage('Check Docker') {
            steps {
                echo "Docker və Docker Compose versiyalarını yoxlayırıq..."
                sh 'docker --version'
                sh "${DOCKER_COMPOSE} --version"
            }
        }

        stage('Checkout') {
            steps {
                echo "Kodu GitHub-dan çəkirik..."
                git url: 'https://github.com/thehajiyev/wordpress-6.4.5.git', branch: 'main'
            }
        }

        stage('Start Services with Docker Compose') {
            steps {
                echo "WordPress və MySQL konteynerlərini qaldırırıq..."
                sh "${DOCKER_COMPOSE} up -d"
                sh 'sleep 20'
            }
        }

        stage('SAST Scan (Semgrep)') {
            steps {
                echo "SAST skanı aparılır (Semgrep)..."
                sh '''
                    pip install semgrep || true
                    semgrep --config=auto --output=semgrep-results.json --json .
                '''
            }
        }

        stage('DAST Scan (OWASP ZAP)') {
            steps {
                echo "DAST skanı aparılır (OWASP ZAP)..."
                sh '''
                    docker run -u zap -p 8090:8090 -d --name zap \
                      owasp/zap2docker-stable zap.sh -daemon -port 8090 -host 0.0.0.0 -config api.disablekey=true
                    sleep 15
                    docker exec zap zap-cli -p 8090 quick-scan --self-contained http://localhost:8000
                    docker exec zap zap-cli report -o zap-report.html -f html
                    docker stop zap
                    docker rm zap
                '''
            }
        }

        stage('Arxivlə') {
            steps {
                echo "Skan nəticələrini saxlayırıq..."
                archiveArtifacts artifacts: '*.json,*.html', allowEmptyArchive: true
            }
        }

        stage('Təmizlik') {
            steps {
                echo "Docker konteynerlərini söndürürük..."
                sh "${DOCKER_COMPOSE} down"
            }
        }
    }

    post {
        always {
            echo 'Pipeline tamamlandı.'
        }
        failure {
            echo 'Pipeline uğursuz oldu!'
        }
    }
}
