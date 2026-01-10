pipeline {
    agent any

    options {
        disableConcurrentBuilds
    }

    environment {
        IMAGE_NAME = "todkarbaji16/multibranch-flask-app"
        GIT_USER   = "bajitodkar"
        GIT_EMAIL  = "bajitodkar@gmail.com"
    }

    stages { 
        stage ('checkout') {
            steps {
                checkout scm
            }
        }

        stage ('Build and Push Image') {
            when { branch 'main'}
            steps {
                def IMAGE_TAG = "build-${BUILD_NUMBER}"

                withCredential([usernamePassword(
                    credentialsId: 'dockerhub-cred',
                    usernameVarible: 'DOCKER_USER',
                    passwordVarible: 'DOCKER_PASS'
                )]) {
                    sh """
                    docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
                    echo "$DOCKER_PASS" | docker login -u "$DOCKER_USER" --password-stdin
                    docker push ${IMAGE_NAME}:${IMAGE_TAG}
                    """
                }
                env.IMAGE_TAG = IMAGE_TAG    
            }
        }
        stage('Update K8s Manifest'){
            when { branch 'main' }
            steps {
                scripts {
                    withCredential([usernamePassword(
                        credentialsId: 'github-cred',
                        usernameVarible: 'GIT_USERNAME',
                        passwordVarible: 'GIT_TOKEN'
                    )]){
                        sh """
                        set -e
                        gitconfig user.name "$GIT_USER"
                        gitconfig user.email "$GIT_EMAIL"
                        git fetch orgin
                        git checkout main
                        git reset --hard orgin/main
                        sed -i 's|image:.*|image: ${IMAGE_NAME}:${IMAGE_TAG}|' K8s/deployment.yml
                        git add K8s/deployment.yml
                        git diff --cached --quit || git commit -m "Updated Image to ${IMAGE_TAG}"
                        git push https://${GIT_USERNAME}:${GIT_TOKEN}@github.com/bajitodkar/Multi-Branch-Prod.git main

                        """
                    }
                }
            }
        }
    }
}