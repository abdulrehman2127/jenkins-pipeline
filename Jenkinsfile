pipeline {
    agent any
    options {
        disableConcurrentBuilds()
    }
    environment {
        IMAGE_NAME = 'abdulrehmanabid/jenkins-pipeline:latest'
        GIT_USER = 'abdulrehman2127'
        GIT_EMAIL = 'abdulrehmanabid2127@gmail.com'
    }

    stages {
        stage('Checkout'){
            checkout scm
        }
        stage('Build') {
           when {branch 'main' }
            steps {
                echo 'Building...'
                def IMAGE_TAG= "build-${BUILD_NUMBER}"
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhubcred',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS')])
                    {
                    sh "docker build -t ${IMAGE_NAME}:${IMAGE_TAG} ."
                    sh "echo ${DOCKERHUB_PASSWORD} | docker login -u ${DOCKERHUB_USERNAME} --password-stdin"
                    sh "docker push ${IMAGE_NAME}:${IMAGE_TAG}"
                }
                env.IMAGE_TAG = IMAGE_TAG
            }
        }
        stage('Update k8s manifests') {
            when{
                branch 'main'
            }
            steps {
                echo 'Deploying...'
                scripts{withCredentials([usernamePassword(
                    credentialsId: 'githubcred',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_TOKEN')]){
                        sh """
                            set -e
                            git config --global user.name "$GIT_USER"
                            git config --global user.email "$GIT_EMAIL"
                            git fetch origin
                            git checkout main
                            git reset --hard origin/main
                            sed -i 's|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|g' k8s/deployment.yml
                            git add k8s/deployment.yml
                            git diff --cached --quiet || (git commit -m "Update deployment image to ${IMAGE_NAME}:${IMAGE_TAG}" && git push origin main)

                        """
                    }}
        }
    }
}