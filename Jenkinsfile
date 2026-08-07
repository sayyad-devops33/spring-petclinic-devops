pipeline {

    agent any

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Maven Build') {
            steps {
                sh './mvnw clean package'
            }
        }

    

        stage('Archive Artifact') {
    steps {
        archiveArtifacts artifacts: 'target/*.jar', fingerprint: true
    }
}

        stage('Docker Build') {
            steps {
                sh "docker build -t spring-petclinic:${BUILD_NUMBER} ."
            }
        }
        stage('Push Docker Image') {
    steps {
        withCredentials([usernamePassword(
            credentialsId: 'dockerhub-creds',
            usernameVariable: 'DOCKER_USER',
            passwordVariable: 'DOCKER_PASS'
        )]) {

            sh '''
                echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin

                docker tag spring-petclinic:${BUILD_NUMBER} $DOCKER_USER/spring-petclinic:${BUILD_NUMBER}

                docker push $DOCKER_USER/spring-petclinic:${BUILD_NUMBER}

                docker logout
            '''
        }
    }
}

    }

    post {

    success {
        echo 'Pipeline completed successfully!'
    }

    failure {
        echo 'Pipeline failed!'
    }

    always {
        echo 'Pipeline execution finished.'
    }

}

}