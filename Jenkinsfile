pipeline {
    agent any

    stages {
        stage('Git Cloning') {
            steps {
                echo 'Cloning git repo'
                git url: 'https://github.com/talhaakts/flask-monitoring.git',  branch: 'main'
            }
        }
        stage('Build Docker Image') {
            steps {
                echo 'Building the image'
                sh 'docker build -t flask-monitoring .'
            }
        }
        stage('Push to Docker Hub') {
            steps {
                echo 'Pushing to Docker Hub'
                withCredentials([usernamePassword(credentialsId: 'dockerhub', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo "${PASS}" | docker login --username "${USER}" --password-stdin
                    docker tag flask-monitoring ${USER}/flask-monitoring:latest
                    docker push ${USER}/flask-monitoring:latest
                    '''
                }
            }
        }
        stage('Trivy Scan') {
            steps {
                 sh 'trivy image talhaakts/flask-monitoring:latest'
                  }
        }
        stage('Integrate Remote kubernetess with Jenkins') {
            steps {
                withCredentials([string(credentialsId: 'secret-token', variable: 'KUBE_TOKEN')]) {
                    sh '''
                    curl -LO "https://storage.googleapis.com/kubernetes-release/release/v1.20.5/bin/linux/amd64/kubectl"
                    chmod u+x ./kubectl
                    export KUBECONFIG=$(mktemp)
                    ./kubectl config set-cluster do-nyc1-k8s-jenkins --server=https://7f55308d-0e33-4797-ad05-2fc0e401c582.k8s.ondigitalocean.com --insecure-skip-tls-verify=true
                    ./kubectl config set-credentials jenkins --token=${KUBE_TOKEN}
                    ./kubectl config set-context default --cluster=do-nyc1-k8s-jenkins --user=jenkins --namespace=default
                    ./kubectl config use-context default
                    ./kubectl get nodes
                    ./kubectl apply -f service.yaml
                    ./kubectl apply -f deployment.yaml
                    '''
                }
            }
        }
    }
}
