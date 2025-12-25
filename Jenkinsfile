pipeline {
    agent any

    environment {
        GITOPS_REPO = "git@github.com:vighneshsikati77/mern-app.git"
        FRONTEND_IMAGE = "vighneshsikati77/frontend"
        BACKEND_IMAGE  = "vighneshsikati77/backend"
        IMAGE_TAG = "${BUILD_NUMBER}"
    }

    stages {

        stage('Prepare SSH') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'github-ssh',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    sh '''
                        mkdir -p ~/.ssh
                        chmod 700 ~/.ssh
                        ssh-keyscan github.com >> ~/.ssh/known_hosts
                        chmod 644 ~/.ssh/known_hosts
                        export GIT_SSH_COMMAND="ssh -i $SSH_KEY -o UserKnownHostsFile=~/.ssh/known_hosts"
                    '''
                }
            }
        }

        stage('Clone GitOps Repo') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'github-ssh',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    sh '''
                        export GIT_SSH_COMMAND="ssh -i $SSH_KEY -o UserKnownHostsFile=~/.ssh/known_hosts"
                        rm -rf gitops-temp
                        git clone -b main $GITOPS_REPO gitops-temp
                    '''
                }
            }
        }

        stage('Update Helm Values') {
            steps {
                sh '''
                    cd gitops-temp/helm/mern-app

                    echo "Updating frontend image tag"
                    sed -i "s|tag:.*|tag: ${IMAGE_TAG}|" values-frontend.yaml

                    echo "Updating backend image tag"
                    sed -i "s|tag:.*|tag: ${IMAGE_TAG}|" values-backend.yaml
                '''
            }
        }

        stage('Commit & Push Changes') {
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'github-ssh',
                        keyFileVariable: 'SSH_KEY'
                    )
                ]) {
                    sh '''
                        export GIT_SSH_COMMAND="ssh -i $SSH_KEY -o UserKnownHostsFile=~/.ssh/known_hosts"
                        cd gitops-temp
                        git config user.email "jenkins@ci.local"
                        git config user.name "Jenkins"

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
            echo "✅ Jenkins pipeline completed. ArgoCD will now sync automatically."
        }
        failure {
            echo "❌ Pipeline failed. Check logs."
        }
    }
}
