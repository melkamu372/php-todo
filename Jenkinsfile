pipeline {
    agent any
    environment {
        PHPUNIT_VERSION = '7.5.20' // Replace with the desired PHPUnit version
    }
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

        stage('Download PHPUnit') {
            steps {
                sh 'wget -O phpunit https://phar.phpunit.de/phpunit-${PHPUNIT_VERSION}.phar'
                sh 'chmod +x phpunit'
            }
        }

        stage('Download PHPLOC') {
            steps {
                sh 'wget -O phploc.phar https://phar.phpunit.de/phploc.phar'
                sh 'chmod +x phploc.phar'
            }
        }

        stage('Run Unit Tests') {
            steps {
                // Execute PHPUnit tests using the downloaded PHPUnit version
                sh './phpunit --configuration phpunit.xml'
            }
        }

        stage('Code Analysis') {
            steps {
                // Execute PHPLOC for code analysis
                sh './phploc.phar app/ --log-csv build/logs/phploc.csv'

                // Archive PHPLOC CSV file as a build artifact
                archiveArtifacts artifacts: 'build/logs/phploc.csv', allowEmptyArchive: true
            }
        }

        stage('Plot Code Coverage Report') {
            steps {
                script {
                    // Plot various metrics from the phploc.csv file
                    plot csvFileName: 'plot-loc.csv', 
                         csvSeries: [[displayTableFlag: false, exclusionValues: 'Lines of Code (LOC),Comment Lines of Code (CLOC),Non-Comment Lines of Code (NCLOC),Logical Lines of Code (LLOC)', 
                                      file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], 
                         group: 'phploc', numBuilds: '100', style: 'line', title: 'A - Lines of code', yaxis: 'Lines of Code'
                    
                    plot csvFileName: 'plot-structures.csv', 
                         csvSeries: [[displayTableFlag: false, exclusionValues: 'Directories,Files,Namespaces', 
                                      file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], 
                         group: 'phploc', numBuilds: '100', style: 'line', title: 'B - Structures Containers', yaxis: 'Count'
                    
                    plot csvFileName: 'plot-average-length.csv', 
                         csvSeries: [[displayTableFlag: false, exclusionValues: 'Average Class Length (LLOC),Average Method Length (LLOC),Average Function Length (LLOC)', 
                                      file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], 
                         group: 'phploc', numBuilds: '100', style: 'line', title: 'C - Average Length', yaxis: 'Average Lines of Code'
                    
                    plot csvFileName: 'plot-cyclomatic-complexity.csv', 
                         csvSeries: [[displayTableFlag: false, exclusionValues: 'Cyclomatic Complexity / Lines of Code,Cyclomatic Complexity / Number of Methods', 
                                      file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], 
                         group: 'phploc', numBuilds: '100', style: 'line', title: 'D - Relative Cyclomatic Complexity', yaxis: 'Cyclomatic Complexity by Structure'
                    
                    plot csvFileName: 'plot-class-types.csv', 
                         csvSeries: [[displayTableFlag: false, exclusionValues: 'Classes,Abstract Classes,Concrete Classes', 
                                      file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], 
                         group: 'phploc', numBuilds: '100', style: 'line', title: 'E - Types of Classes', yaxis: 'Count'
                    
                    plot csvFileName: 'plot-method-types.csv', 
                         csvSeries: [[displayTableFlag: false, exclusionValues: 'Methods,Non-Static Methods,Static Methods,Public Methods,Non-Public Methods', 
                                      file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], 
                         group: 'phploc', numBuilds: '100', style: 'line', title: 'F - Types of Methods', yaxis: 'Count'
                    
                    plot csvFileName: 'plot-constants-types.csv', 
                         csvSeries: [[displayTableFlag: false, exclusionValues: 'Constants,Global Constants,Class Constants', 
                                      file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], 
                         group: 'phploc', numBuilds: '100', style: 'line', title: 'G - Types of Constants', yaxis: 'Count'
                    
                    plot csvFileName: 'plot-testing.csv', 
                         csvSeries: [[displayTableFlag: false, exclusionValues: 'Test Classes,Test Methods', 
                                      file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], 
                         group: 'phploc', numBuilds: '100', style: 'line', title: 'I - Testing', yaxis: 'Count'
                    
                    plot csvFileName: 'plot-code-structure.csv', 
                         csvSeries: [[displayTableFlag: false, exclusionValues: 'Logical Lines of Code (LLOC),Classes Length (LLOC),Functions Length (LLOC),LLOC outside functions or classes', 
                                      file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], 
                         group: 'phploc', numBuilds: '100', style: 'line', title: 'AB - Code Structure by Logical Lines of Code', yaxis: 'Logical Lines of Code'
                    
                    plot csvFileName: 'plot-functions-types.csv', 
                         csvSeries: [[displayTableFlag: false, exclusionValues: 'Functions,Named Functions,Anonymous Functions', 
                                      file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], 
                         group: 'phploc', numBuilds: '100', style: 'line', title: 'H - Types of Functions', yaxis: 'Count'
                    
                    plot csvFileName: 'plot-structure-objects.csv', 
                         csvSeries: [[displayTableFlag: false, exclusionValues: 'Interfaces,Traits,Classes,Methods,Functions,Constants', 
                                      file: 'build/logs/phploc.csv', inclusionFlag: 'INCLUDE_BY_STRING', url: '']], 
                         group: 'phploc', numBuilds: '100', style: 'line', title: 'BB - Structure Objects', yaxis: 'Count'
                }
            }
        }

        stage('Package Artifact') {
            steps {
                sh 'zip -qr php-todo.zip ${WORKSPACE}/*'
                archiveArtifacts artifacts: 'php-todo.zip', allowEmptyArchive: false
            }
        }

        stage('SonarQube Quality Gate') {
            when { branch pattern: "^develop*|^hotfix*|^release*|^main*", comparator: "REGEXP" }
            environment {
                scannerHome = tool 'SonarQubeScanner'
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh "${scannerHome}/bin/sonar-scanner -Dproject.settings=sonar-project.properties"
                }
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    
        // stage('Upload to Artifactory') {
        //     steps {
        //         script {
        //             def server = Artifactory.server 'my-artifactory-server'
        //             def uploadSpec = """{
        //                 "files": [
        //                     {
        //                         "pattern": "php-todo.zip",
        //                         "target": "libs-release-local/php-todo/${env.BUILD_NUMBER}/"
        //                     }
        //                 ]
        //             }"""
        //             server.upload(uploadSpec)
        //         }
        //     }
        // }
        stage ('Deploy to Dev Environment') {
            // agent { label 'slave_one' }
            steps {
                build job: 'ansible-config-mgt/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
            }
        }

        stage ('Deploy to Test Environment') {
            // agent { label 'slave_two' } 
            steps {
                build job: 'ansible-config-mgt/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
            }
        }

        stage ('Deploy to Production Environment') {
            // agent any
            steps {
                build job: 'ansible-config-mgt/main', parameters: [[$class: 'StringParameterValue', name: 'env', value: 'dev']], propagate: false, wait: true
            }
        }
    }
}
