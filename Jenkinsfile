pipeline {
    agent any

    stages {

        stage('Check Tools') {
            steps {
                sh '''
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
                sh 'docker build -t salary-api:v2 .'
            }
        }

        stage('Trivy Image Scan') {
            steps {
                sh 'trivy image --severity HIGH,CRITICAL salary-api:v2'
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

                    docker tag salary-api:v2 $DOCKER_USER/salary-api:v2

                    docker push $DOCKER_USER/salary-api:v2
                    '''
                }
            }
        }
    }
}
