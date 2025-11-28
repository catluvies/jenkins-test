pipeline {
    agent any

    environment {
        PROJECT_NAME = "pipeline-test"
        SONARQUBE_URL = "http://sonarqube:9000"
        SONARQUBE_TOKEN = "sqa_913376b5aa2d32bebbd697018145b89c7f76116e"
        TARGET_URL = "http://172.19.52.116:5000"
    }

    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                    python3 -m pip install --upgrade pip --break-system-packages
                    pip3 install --break-system-packages -r requirements.txt
                '''
            }
        }

        stage('OWASP Dependency-Check') {
            steps {
                dependencyCheck additionalArguments: """
                    --scan .
                    --format HTML
                    --format JSON
                    --nvdApiKey ${env.NVD_API_KEY}
                    --enableExperimental
                """, odcInstallation: 'DependencyCheck'
                
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('SonarQubeScanner') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                            -Dsonar.projectKey=${PROJECT_NAME} \
                            -Dsonar.sources=. \
                            -Dsonar.host.url=${SONARQUBE_URL} \
                            -Dsonar.token=${SONARQUBE_TOKEN}
                        """
                    }
                }
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: '**/dependency-check-report.*', allowEmptyArchive: true
        }
    }
}
