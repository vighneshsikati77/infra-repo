pipeline {
    agent any

    environment {
        GITOPS_REPO = "git@github.com:vighneshsikati77/mern-app.git"
        IMAGE_TAG = "${BUILD_NUMBER}" // Use Jenkins build number
        BACKEND_IMAGE = "vighneshsikati77/mern-backend"
        FRONTEND_IMAGE = "vighneshsikati77/mern-frontend"
        DOCKERHUB_CREDENTIALS = credentials('dockerhub-creds')
    }

    stages {

        stage('Prepare SSH') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'github-ssh', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        mkdir -p ~/.ssh
                        chmod 700 ~/.ssh
                        ssh-keyscan github.com >> ~/.ssh/known_hosts
                        chmod 644 ~/.ssh/known_hosts
                    '''
                }
            }
        }

        stage('Checkout Code') {
            steps {
                git branch: 'main', url: 'https://github.com/vighneshsikati77/infra-repo.git'
            }
        }

        stage('Build Backend Image') {
            steps {
                dir('backend') {
                    sh "docker build -t $BACKEND_IMAGE:$IMAGE_TAG ."
                }
            }
        }

        stage('Build Frontend Image') {
            steps {
                dir('frontend') {
                    sh "docker build -t $FRONTEND_IMAGE:$IMAGE_TAG ."
                }
            }
        }

        stage('DockerHub Login') {
            steps {
                sh '''
                    echo $DOCKERHUB_CREDENTIALS_PSW | docker login -u $DOCKERHUB_CREDENTIALS_USR --password-stdin
                '''
            }
        }

        stage('Push Images') {
            steps {
                sh '''
                    docker push $BACKEND_IMAGE:$IMAGE_TAG
                    docker push $FRONTEND_IMAGE:$IMAGE_TAG
                '''
            }
        }

        stage('Clone GitOps Repo') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'github-ssh', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        export GIT_SSH_COMMAND="ssh -i $SSH_KEY -o UserKnownHostsFile=~/.ssh/known_hosts"
                        rm -rf gitops-temp
                        git clone -b main $GITOPS_REPO gitops-temp
                    '''
                }
            }
        }

        stage('Update Helm values.yaml') {
            steps {
                sh '''
                    echo "Helm directories:"
                    ls gitops-temp/helm

                    echo "Updating FRONTEND image tag"
                    sed -i "s/tag:.*/tag: ${IMAGE_TAG}/" gitops-temp/helm/frontend/values.yaml

                    echo "Updating BACKEND image tag"
                    sed -i "s/tag:.*/tag: ${IMAGE_TAG}/" gitops-temp/helm/backend/values.yaml

                    echo "Updating MONGODB image tag"
                    sed -i "s|^\\(\\s*tag:\\).*|\\1 6.0|" gitops-temp/helm/mongodb/values.yaml
                '''
            }
        }

        stage('Commit & Push Helm Changes') {
            steps {
                withCredentials([sshUserPrivateKey(credentialsId: 'github-ssh', keyFileVariable: 'SSH_KEY')]) {
                    sh '''
                        export GIT_SSH_COMMAND="ssh -i $SSH_KEY -o UserKnownHostsFile=~/.ssh/known_hosts"
                        cd gitops-temp
                        git config user.email "vighneshsiakti77@gmail.com"
                        git config user.name "vighneshsikati77"
                        git add .
                        git commit -m "Update image tags to ${IMAGE_TAG}" || echo "No changes to commit"
                        git push origin main
                    '''
                }
            }
        }
    }

    post {
        success {
            echo "✅ Images pushed and Helm values updated. ArgoCD will auto-deploy."
        }
        failure {
            echo "❌ Pipeline failed."
        }
    }
}
