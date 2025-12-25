pipeline {
    agent any

    environment {
        # GitOps repo containing Helm charts & ArgoCD manifests
        GITOPS_REPO = 'https://github.com/vighneshsikati77/mern-app.git'
        GITOPS_BRANCH = 'main'
    }

    stages {

        stage('Clone GitOps Repo') {
            steps {
                echo "Cloning GitOps repo..."
                sh "rm -rf gitops-temp"
                sh "git clone -b ${GITOPS_BRANCH} ${GITOPS_REPO} mern-app"
            }
        }

        stage('Update Helm values.yaml for Backend') {
            steps {
                echo "Updating backend image in Helm chart..."
                sh """
                sed -i 's|repository:.*|repository: vighneshsikati77/backend|' mern-app/helm/backend/values.yaml
                sed -i 's|tag:.*|tag: latest|' mern-app/helm/backend/values.yaml
                """
            }
        }

        stage('Update Helm values.yaml for Frontend') {
            steps {
                echo "Updating frontend image in Helm chart..."
                sh """
                sed -i 's|repository:.*|repository: vighneshsikati77/frontend|' mern-appp/helm/frontend/values.yaml
                sed -i 's|tag:.*|tag: latest|' mern-app/helm/frontend/values.yaml
                """
            }
        }

        stage('Push Changes to GitOps Repo') {
            steps {
                echo "Committing and pushing changes to GitOps repo..."
                sh """
                cd gitops-temp
                git config --global user.email "vighneshsikati77@gmail.com"
                git config --global user.name "vighneshsikati77"
                git add .
                git commit -m "Update image tags from Jenkins"
                git push origin ${GITOPS_BRANCH}
                """
            }
        }
    }

    post {
        success {
            echo "✅ Jenkins pipeline completed successfully!"
        }
        failure {
            echo "❌ Jenkins pipeline failed!"
        }
    }
}
