pipeline {
    agent any

    stages {
        stage('Kodu GitHub-dan çek') {
            steps {
                git branch: 'main', url: 'https://github.com/thehajiyev/wordpress-6.4.5.git'
            }
        }

        stage('Docker ilə WordPress işə sal') {
            steps {
                sh '''
                docker run -d --rm --name wp-pipeline \
                -v $WORKSPACE:/var/www/html \
                -p 8085:80 \
                wordpress:php8.1-apache
                '''
            }
        }
    }
}

