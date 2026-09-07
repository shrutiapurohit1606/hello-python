pipeline {
    agent any

    environment {
        SONARQUBE = 'sonarqube'
        SONAR_TOKEN = credentials('sonar-token')
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
                    pytest -q
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'

                    withSonarQubeEnv('sonarqube') {
                        sh """
                            ${scannerHome}/bin/sonar-scanner \
                              -Dsonar.projectKey=hello-python \
                              -Dsonar.sources=. \
                              -Dsonar.host.url=http://34.173.27.184:9000 \
                              -Dsonar.token=\\$SONAR_TOKEN
                        """
                    }
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage('Deploy') {
            steps {
                sshagent(credentials: ['gce-ssh']) {
                    sh '''
                        ssh -o StrictHostKeyChecking=no shrutiapurohit@10.128.0.8 '
                            mkdir -p ~/app &&
                            cd ~/app &&
                            git clone -q https://github.com/shrutiapurohit1606/hello-python.git . 2>/dev/null || git pull -q &&
                            python3 -m pip install --user -r requirements.txt &&
                            pkill -f "python3 app.py" || true &&
                            nohup python3 app.py > app.log 2>&1 &
                        '
                    '''
                }
            }
        }
    }
}
