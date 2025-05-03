pipeline {
    agent any

    environment {
        DOCKER_IMAGE = 'laravel-app'
        DOCKER_TAG = 'latest'
    }

    stages {
        stage('Checkout Code') {
            steps {
                git credentialsId: 'github-token', url: 'https://github.com/prashanty3/laraveldevops.git'
            }
        }

        stage('Composer Install') {
            steps {
                sh 'composer install --no-interaction'
                sh 'cp .env.example .env'
                sh 'php artisan key:generate'
            }
        }

        stage('Validate Code') {
            steps {
                // Syntax check
                sh 'find . -name "*.php" -print0 | xargs -0 -n1 php -l'
                
                // Optional - Code style
                sh 'vendor/bin/pint --test || true'

                // Optional - Static analysis
                sh 'vendor/bin/phpstan analyse || true'
            }
        }

        stage('Security Check') {
            steps {
                sh 'composer require --dev enlightn/security-checker'
                sh 'vendor/bin/security-checker security:check'
            }
        }

        stage('Run Tests') {
            steps {
                sh 'php artisan test'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh 'docker build -t ${DOCKER_IMAGE}:${DOCKER_TAG} .'
            }
        }

        stage('Run Docker Container') {
            steps {
                sh 'docker run -d -p 8000:80 --name test-container ${DOCKER_IMAGE}:${DOCKER_TAG}'
            }
        }

        stage('Clean Up') {
            steps {
                sh 'docker stop test-container || true'
                sh 'docker rm test-container || true'
            }
        }
    }

    post {
        always {
            echo 'Pipeline execution finished.'
        }
        failure {
            echo 'Build failed.'
        }
    }
}
