pipeline {
    agent any

    environment {
        COMPOSE_PROJECT_NAME = "wordpress_test"
    }

    stages {
        stage('Checkout') {
            steps {
                git url: 'https://github.com/thehajiyev/wordpress-6.4.5.git', branch: 'main'
            }
        }

        stage('Docker Compose Up') {
            steps {
                echo "WordPress və MySQL konteynerlərini qaldırırıq..."
                sh 'docker-compose up -d'
                sh 'sleep 20' // Servis yuxarı qalxana qədər gözləyək
            }
        }

        stage('SAST Scan (Semgrep)') {
            steps {
                echo "SAST skanı aparılır..."
                sh '''
                pip install semgrep || true
                semgrep --config=auto --output=semgrep-results.json --json .
                '''
            }
        }

        stage('DAST Scan (OWASP ZAP)') {
            steps {
                echo "DAST skanı OWASP ZAP ilə aparılır..."
                sh '''
                docker run -u zap -p 8090:8090 -d --name zap \
                  owasp/zap2docker-stable zap.sh -daemon -port 8090 -host 0.0.0.0 -config api.disablekey=true
                sleep 10
                docker exec zap zap-cli -p 8090 quick-scan --self-contained http://localhost:8000
                docker exec zap zap-cli report -o zap-report.html -f html
                docker stop zap
                docker rm zap
                '''
            }
        }

        stage('Archive Reports') {
            steps {
                archiveArtifacts artifacts: '*.json,*.html', allowEmptyArchive: true
            }
        }

        stage('Docker Compose Down') {
            steps {
                echo "Təmizləmə: konteynerləri söndürürük"
                sh 'docker-compose down'
            }
        }
    }

    post {
        always {
            echo 'Pipeline tamamlandı (uğurlu və ya uğursuz)'
        }
        failure {
            echo 'Pipeline uğursuz oldu!'
        }
    }
}
