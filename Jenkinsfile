pipeline {
    agent any

    stages {

        stage('Check Tools') {
            steps {
                sh '''
                echo "===== OWASP ZAP ====="

                echo "Checking ZAP Binary..."
                zap.sh -version || true

                echo "Checking zap-baseline.py..."
                zap-baseline.py -h || true

                echo "Checking ZAP Path..."
                which zap.sh || true

                echo "Searching zap-baseline.py..."
                find / -name "zap-baseline.py" 2>/dev/null || true

                echo "Listing ZAP Installation Directory..."
                ls -l $(dirname $(which zap.sh)) || true

                echo "===== Docker ====="
                docker --version || true

                echo "===== Kubectl ====="
                kubectl version --client || true

                echo "===== AWS CLI ====="
                aws --version || true

                echo "===== Eksctl ====="
                eksctl version || true
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

        stage('Check Application') {
            steps {
                sh '''
                echo "===== Checking Application ====="

                curl -I http://13.203.21.160:30080/actuator/health
                '''
            }
        }
    }
}
