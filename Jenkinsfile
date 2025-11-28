pipeline {
    agent any
    environment {
        PROJECT_NAME = "pipeline-test"
        SONARQUBE_URL = "http://sonarqube:9000"
        SONARQUBE_TOKEN = "sqa_913376b5aa2d32bebbd697018145b89c7f76116e"
        TARGET_URL = "http://172.19.52.116:5000"
        ZAP_API_KEY = "zap123"
    }
    stages {
        stage('Install Python') {
            steps {
                sh '''
                    apt update
                    apt install -y python3 python3-venv python3-pip curl
                '''
            }
        }
        
        stage('Setup Environment') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }
        
        stage('Python Security Audit') {
            steps {
                sh '''
                    . venv/bin/activate
                    pip install pip-audit
                    mkdir -p dependency-check-report
                    pip-audit -r requirements.txt -f markdown -o dependency-check-report/pip-audit.md || true
                '''
            }
        }
        
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarQubeScanner'
                    withSonarQubeEnv('SonarQubeScanner') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                                -Dsonar.projectKey=$PROJECT_NAME \
                                -Dsonar.sources=. \
                                -Dsonar.host.url=$SONARQUBE_URL \
                                -Dsonar.login=$SONARQUBE_TOKEN
                        """
                    }
                }
            }
        }
        
        stage('OWASP ZAP Scan') {
            steps {
                sh '''
                    mkdir -p zap-reports
                    
                    echo "Iniciando spider scan..."
                    curl "http://zap:8090/JSON/spider/action/scan/?url=${TARGET_URL}&apikey=${ZAP_API_KEY}"
                    sleep 30
                    
                    echo "Iniciando active scan..."
                    curl "http://zap:8090/JSON/ascan/action/scan/?url=${TARGET_URL}&apikey=${ZAP_API_KEY}"
                    sleep 60
                    
                    echo "Generando reporte..."
                    curl "http://zap:8090/OTHER/core/other/htmlreport/?apikey=${ZAP_API_KEY}" > zap-reports/zap-report.html
                '''
            }
        }
        
        stage('Dependency Check') {
            environment {
                NVD_API_KEY = credentials('nvdApiKey')
            }
            steps {
                dependencyCheck additionalArguments: "--scan . --format HTML --out dependency-check-report --enableExperimental --enableRetired --nvdApiKey ${NVD_API_KEY}", odcInstallation: 'DependencyCheck'
            }
        }
        
        stage('Publish Reports') {
            steps {
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'dependency-check-report',
                    reportFiles: 'dependency-check-report.html',
                    reportName: 'OWASP Dependency Check Report'
                ])
                publishHTML([
                    allowMissing: false,
                    alwaysLinkToLastBuild: true,
                    keepAll: true,
                    reportDir: 'zap-reports',
                    reportFiles: 'zap-report.html',
                    reportName: 'OWASP ZAP Report'
                ])
            }
        }
    }
}
