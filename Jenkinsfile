pipeline {
    agent any

    options {
        disableConcurrentBuilds()
    }

    stages {
        stage('Recuperer le code') {
            steps {
                git branch: 'main',
                    url: 'https://github.com/Eyahagit/4SSA11-hamdiEya.git'
            }
        }

        stage('Compiler et tester') {
            steps {
                dir('backend') {
                    sh 'mvn -B clean install'
                }
            }
        }

        stage('Analyse SonarQube') {
            steps {
                withCredentials([
                    string(credentialsId: 'sonar-token',
                           variable: 'SONAR_TOKEN')
                ]) {
                    dir('backend') {
                        sh '''
                            mvn -B org.sonarsource.scanner.maven:sonar-maven-plugin:sonar \
                              -Dsonar.projectKey=4SSA11-hamdiEya \
                              -Dsonar.host.url=http://localhost:9000
                        '''
                    }
                }
            }
        }

        stage('Archiver le livrable') {
            steps {
                archiveArtifacts artifacts: 'backend/target/*.jar',
                    fingerprint: true
            }
        }
    }

    post {
        always {
            junit testResults: 'backend/target/surefire-reports/*.xml',
                allowEmptyResults: true
        }
    }
}









