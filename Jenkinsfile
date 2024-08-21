pipeline {
    agent any

    stages {
        stage('Git Cloning') {
            steps {
                echo 'Cloning git repo'
                git url: 'https://github.com/hakanbayraktar/flask-monitoring.git', credentialsId: 'jenkins-github', branch: 'main'
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
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'USER', passwordVariable: 'PASS')]) {
                    sh '''
                    echo "${PASS}" | docker login --username "${USER}" --password-stdin
                    docker tag flask-monitoring ${USER}/flask-monitoring:latest
                    docker push ${USER}/flask-monitoring:latest
                    '''
                }
            }
        }
        stage('Kubernetes Deploy') {
            steps {
                echo 'Deploying to Kubernetes...'
                withCredentials([file(credentialsId: 'kubernetes-cred', variable: 'KUBECONFIG')]) {
                    sh '''
                    kubectl --kubeconfig=${KUBECONFIG} apply -f deployment.yaml
                    kubectl --kubeconfig=${KUBECONFIG} apply -f service.yaml
                    '''
                }
            }
        }
    }
}
