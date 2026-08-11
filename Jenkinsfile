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
            #!/bin/bash
            
            PREVIOUS_BUILD=$((BUILD_NUMBER - 1))

            echo "Current build: ${BUILD_NUMBER}"
            echo "Previous build: ${PREVIOUS_BUILD}"

            docker pull ${DOCKER_HUB_REPO}:${BUILD_NUMBER}

            docker stop petclinic || true

            docker rm petclinic || true

            docker run -d \
              --name petclinic \
              -p 8081:8080 \
              ${DOCKER_HUB_REPO}:${BUILD_NUMBER}

            echo "Waiting for application to become healthy..."

            healthy=false

                for i in $(seq 1 12); do
                if curl -f -s http://localhost:8081 > /dev/null; then
                    echo "Application is healthy on attempt $i!"
                    healthy=true
                    break
                fi

                echo "Health check attempt $i failed. Waiting 5 seconds..."
                sleep 5
            done
        if [ "$healthy" = "true" ]; then
    echo "Deployment successful!"
else
    echo "Deployment failed. Starting rollback..."

    docker stop petclinic || true
    docker rm petclinic || true

    echo "Rolling back to build ${PREVIOUS_BUILD}..."

    docker run -d \
      --name petclinic \
      -p 8081:8080 \
      ${DOCKER_HUB_REPO}:${PREVIOUS_BUILD}

    echo "Checking rollback health..."

    rollback_healthy=false

    for i in $(seq 1 12); do
        if curl -f -s http://localhost:8081 > /dev/null; then
            echo "Rollback health check successful on attempt $i!"
            rollback_healthy=true
            break
        fi

        echo "Rollback health check attempt $i failed. Waiting 5 seconds..."
        sleep 5
    done

    if [ "$rollback_healthy" = "true" ]; then
        echo "Rollback successful!"
        exit 1
    else
        echo "CRITICAL: Rollback also failed!"
        exit 1
    fi
fi
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