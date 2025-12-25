pipeline {
    agent any

    environment {
        GITOPS_REPO = "git@github.com:vighneshsikati77/mern-app.git"
    }

    stages {

        stage('Clone GitOps Repo') {
            steps {
                sshagent(credentials: ['github-ssh']) {
                    sh '''
                        mkdir -p ~/.ssh
                        chmod 700 ~/.ssh

                        ssh-keyscan github.com >> ~/.ssh/known_hosts
                        chmod 644 ~/.ssh/known_hosts

                        rm -rf gitops-temp
                        git clone -b main ${GITOPS_REPO} gitops-temp
                    '''
                }
            }
        }

        stage('Update Helm values.yaml for Backend') {
            steps {
                sh '''
                  cd gitops-temp/helm/mern-app/backend
                  sed -i "s|tag:.*|tag: ${BUILD_NUMBER}|" values.yaml
                '''
            }
        }

        stage('Update Helm values.yaml for Frontend') {
            steps {
                sh '''
                  cd gitops-temp/helm/mern-app/frontend
                  sed -i "s|tag:.*|tag: ${BUILD_NUMBER}|" values.yaml
                '''
            }
        }

        stage('Push Changes to GitOps Repo') {
            steps {
                sshagent(credentials: ['github-ssh']) {
                    sh '''
                        cd gitops-temp
                        git config user.email "jenkins@ci.local"
                        git config user.name "Jenkins"

                        git add .
                        git commit -m "Update image tags to ${BUILD_NUMBER}"
                        git push origin main
                    '''
                }
            }
        }
    }

    post {
        failure {
            echo "❌ Jenkins pipeline failed!"
        }
        success {
            echo "✅ GitOps update successful!"
        }
    }
}
