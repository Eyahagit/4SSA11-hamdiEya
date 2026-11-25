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
                    string(
                        credentialsId: 'sonar-token',
                        variable: 'SONAR_TOKEN'
                    )
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

        stage('Construire et deployer Docker') {
            steps {
                dir('backend') {
                    sh '''
                        set -eu

                        docker_config=$(mktemp -d)
                        trap 'rm -rf "$docker_config"' EXIT

                        docker --config "$docker_config" build \
                          -t hamdieya_4ssa11_gestionprojets:jenkins .

                        if docker container inspect devops-backend >/dev/null 2>&1; then
                            docker rm -f devops-backend
                        fi

                        docker run -d --name devops-backend \
                          --network devops-network \
                          -p 127.0.0.1:8082:8081 \
                          -e SPRING_DATASOURCE_URL='jdbc:mysql://devops-mysql:3306/test_db?createDatabaseIfNotExist=true' \
                          -e SPRING_DATASOURCE_USERNAME=root \
                          -e SPRING_DATASOURCE_PASSWORD=root \
                          -e MANAGEMENT_ENDPOINTS_WEB_EXPOSURE_INCLUDE=health,info,prometheus \
                          hamdieya_4ssa11_gestionprojets:jenkins
                    '''
                }
            }
        }

        stage('Verifier le deploiement') {
            steps {
                sh '''
                    python3 - <<'PY'
import time
import urllib.request

url = "http://localhost:8082/actuator/health"

for attempt in range(30):
    try:
        with urllib.request.urlopen(url, timeout=3) as response:
            import json
            health = json.load(response)
            if response.status == 200 and health.get("status") == "UP":
                print("Backend deploye et operationnel")
                break
    except Exception:
        pass
    time.sleep(3)
else:
    raise SystemExit("Le backend ne repond pas correctement")
PY
                '''
            }
        }

        stage('Archiver le livrable') {
            steps {
                archiveArtifacts(
                    artifacts: 'backend/target/*.jar',
                    fingerprint: true
                )
            }
        }
    }

    post {
        always {
            junit(
                testResults: 'backend/target/surefire-reports/*.xml',
                allowEmptyResults: true
            )
        }
    }
}