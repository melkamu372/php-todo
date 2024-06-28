pipeline {
    agent any

    stages {
        stage("Initial Cleanup") {
            steps {
                deleteDir() // Clean up workspace before starting
            }
        }

        stage('Checkout SCM') {
            steps {
                git branch: 'main', url: 'https://github.com/melkamu372/php-todo.git'
            }
        }

        stage('Prepare Environment') {
            steps {
                dir("${WORKSPACE}") {
                    // Rename .env.sample to .env
                    sh 'mv .env.sample .env'

                    // Install Composer dependencies
                    sh 'composer install'

                    // Run Laravel migrations and seed the database
                    sh 'php artisan migrate --force'
                    sh 'php artisan db:seed --force'

                    // Generate application key if not already set
                    sh 'php artisan key:generate'
                }
            }
        }

        stage('Setup Laravel Directories') {
            steps {
                dir("${WORKSPACE}") {
                    // Ensure Laravel storage directories exist
                    sh 'mkdir -p storage/framework/sessions'
                    sh 'mkdir -p storage/framework/cache'
                    sh 'chmod -R 777 storage' // Adjust permissions as needed for your environment
                }
            }
        }

        stage('Run Unit Tests') {
            steps {
                // Execute PHPUnit tests
                sh './vendor/bin/phpunit'
            }
        }
    }

   
}
