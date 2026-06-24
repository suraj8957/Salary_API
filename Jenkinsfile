stage('Docker Build') {
    steps {
        sh 'docker build -t salary-api:v3 .'
    }
}

stage('Trivy Image Scan') {
    steps {
        sh 'trivy image --severity HIGH,CRITICAL salary-api:v3'
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

            docker tag salary-api:v3 $DOCKER_USER/salary-api:v3

            docker push $DOCKER_USER/salary-api:v3
            '''
        }
    }
}
