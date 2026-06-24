pipeline {
    agent any

    environment {
        SONAR_SERVER = 'sonarqube'
        SONAR_TOKEN  = credentials('sonar-token')
    }

    stages {

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

        stage('Unit Test') {
            steps {
                sh 'mvn test'
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONAR_SERVER}") {
                    sh """
                    mvn sonar:sonar \
                    -Dsonar.projectKey=salary-api \
                    -Dsonar.token=${SONAR_TOKEN}
                    """
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
                sh 'trivy fs .'
            }
        }
    }
}
