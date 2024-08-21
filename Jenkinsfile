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
                echo 'pushing to Docker Hub'
                withCredentials([usernamePassword(credentialsId: "dockerhub-cred", usernameVariable: "USER", passwordVariable: "PASS")]) {
                    sh "echo ${PASS} | docker login --username ${USER} --password-stdin"
                    sh "docker tag flask-monitoring ${USER}/flask-monitoring:latest"
                    sh "docker push ${USER}/flask-monitoring:latest"
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
