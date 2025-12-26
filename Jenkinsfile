pipeline {
    agent any

    environment {
        GITOPS_REPO = "git@github.com:vighneshsikati77/mern-app.git"
        IMAGE_TAG = "latest" // or the tag of your DockerHub image
    }

    stages {

        stage('Prepare SSH') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'github-ssh', keyFileVariable: 'SSH_KEY')
                ]) {
                    sh '''
                        mkdir -p ~/.ssh
                        chmod 700 ~/.ssh
                        ssh-keyscan github.com >> ~/.ssh/known_hosts
                        chmod 644 ~/.ssh/known_hosts
                    '''
                }
            }
        }

        stage('Clone GitOps Repo') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'github-ssh', keyFileVariable: 'SSH_KEY')
                ]) {
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
                    echo "Updating Helm values with pre-built Docker images"
                    sed -i "s/tag:.*/tag: ${IMAGE_TAG}/" gitops-temp/helm/frontend/values.yaml
                    sed -i "s/tag:.*/tag: ${IMAGE_TAG}/" gitops-temp/helm/backend/values.yaml
                    sed -i "s|^\\(\\s*tag:\\).*|\\1 6.0|" gitops-temp/helm/mongodb/values.yaml
                '''
            }
        }

        stage('Commit & Push Changes') {
            steps {
                withCredentials([
                    sshUserPrivateKey(credentialsId: 'github-ssh', keyFileVariable: 'SSH_KEY')
                ]) {
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
            echo "✅ Helm values updated. ArgoCD will auto-deploy."
        }
        failure {
            echo "❌ Pipeline failed."
        }
    }
}
