pipeline {

    agent any

    environment {

        APP_NAME = 'spring-petclinic'

        DOCKER_HUB_REPO = 'sayyaddevops33/spring-petclinic'

    }

     options {

        timestamps()

        buildDiscarder(logRotator(
            numToKeepStr: '10'
        ))

        timeout(time: 30, unit: 'MINUTES')

    }

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
                sh "docker build -t ${APP_NAME}:${BUILD_NUMBER} ."
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

                docker tag ${APP_NAME}:${BUILD_NUMBER} ${DOCKER_HUB_REPO}:${BUILD_NUMBER}

                docker push ${DOCKER_HUB_REPO}:${BUILD_NUMBER}

                docker logout
            '''
        }
    }
}

stage('Deploy') {
    steps {
        sh '''
            docker pull ${DOCKER_HUB_REPO}:${BUILD_NUMBER}

            docker stop petclinic || true

            docker rm petclinic || true

            docker run -d \
              --name petclinic \
              -p 8081:8080 \
              ${DOCKER_HUB_REPO}:${BUILD_NUMBER}
        '''
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
        junit 'target/surefire-reports/*.xml'
        cleanWs()
        echo 'Pipeline execution finished.'
    }

}

}