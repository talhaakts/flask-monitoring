pipeline {
    agent any
    stages {
        stage('git cloning') {
            steps {
                echo 'Cloning git repo'
                git url: 'https://github.com/hakanbayraktar/flask-monitoring.git', credentialsId: 'jenkins-github', branch: 'main'

            }
        }
        stage('build') {
            steps {
                echo 'Building the image'
                sh 'docker build -t flask-monitoring .'
            }
        }
        stage('dockerhub') {
            steps {
                echo 'Pushing to Docker Hub'
                withCredentials([usernamePassword(credentialsId: 'dockerhub-cred', usernameVariable: 'user', passwordVariable: 'pass')]) {
                    sh 'echo "$pass" | docker login --username "$user" --password-stdin'
                    sh 'docker tag flask-monitoring "$user"/flask-monitoring:latest'
                    sh 'docker push "$user"/flask-monitoring:latest'
                }
            }
        }


        stage('kubernetes deploy') {
            steps {
                echo 'Deploying to Kubernetes...'
                kubernetesDeploy(
                    configs: 'deployment.yaml,service.yaml',
                    kubeconfigId: 'kubernetes-cred'
                )
            }
        }
    }
}
