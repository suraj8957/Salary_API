pipeline {
    agent any

    stages {

        stage('Check Tools') {
            steps {
                sh '''
                echo "===== OWASP ZAP ====="

                zap.sh -version || true
                which zap.sh || true

                echo "===== Docker ====="
                docker --version || true

                echo "===== AWS CLI ====="
                aws --version || true
                '''
            }
        }

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Compile') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    withCredentials([string(credentialsId: 's-sonar-token', variable: 'SONAR_TOKEN')]) {
                        sh '''
                        mvn sonar:sonar \
                        -Dsonar.projectKey=salary-api \
                        -Dsonar.login=$SONAR_TOKEN
                        '''
                    }
                }
            }
        }

        stage('Package') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Trivy Dependency Scan') {
            steps {
                sh 'trivy fs --severity HIGH,CRITICAL .'
            }
        }

        stage('Docker Build') {
            steps {
                sh 'docker build -t salary-api:v4 .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL salary-api:v4'
            }
        }

        stage('Docker Push') {
            steps {
                withCredentials([
                    usernamePassword(
                        credentialsId: 's-dockerhub-token',
                        usernameVariable: 'DOCKER_USER',
                        passwordVariable: 'DOCKER_PASS'
                    )
                ]) {
                    sh '''
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                    docker tag salary-api:v4 $DOCKER_USER/salary-api:v4

                    docker push $DOCKER_USER/salary-api:v4
                    '''
                }
            }
        }

        stage('DAST') {
            steps {
                sh '''
                echo "===== Running OWASP ZAP Baseline Scan ====="

                mkdir -p zap-report
                chmod -R 777 zap-report

                docker pull ghcr.io/zaproxy/zaproxy:stable

                docker run --rm \
                --user root \
                -v "$PWD/zap-report:/zap/wrk" \
                ghcr.io/zaproxy/zaproxy:stable \
                zap-baseline.py \
                -t http://13.203.21.160:30080 \
                -r zap-report.html \
                -I || true

                chmod -R 777 zap-report || true

                echo "===== Report Files ====="
                ls -lah zap-report || true
                '''
            }
        }
    }

    post {
        always {
            publishHTML(target: [
                allowMissing: true,
                alwaysLinkToLastBuild: true,
                keepAll: true,
                reportDir: 'zap-report',
                reportFiles: 'zap-report.html',
                reportName: 'OWASP ZAP Report'
            ])
        }
    }
}
