pipeline {
    agent any
    stages {
        stage('git cloning') {
            steps {
                echo 'Cloning git repo'
                git url: 'https://github.com/hbayraktar/flask-monitoring.git', branch: 'main'
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
                    sh 'docker login -u ${env.user} -p ${env.pass}'
                    sh 'docker tag flask-monitoring ${env.user}/flask-monitoring:latest'
                    sh 'docker push ${env.user}/flask-monitoring:latest'
                }
            }
        }
        stage('kubernetes deploy') {
            steps {
                echo 'Deploying to Kubernetes...'
                kubernetesDeploy(
                    configs: 'deployment.yaml,service.yaml',
                    kubeconfigId: 'k8s-cluster-cred'
                )
            }
        }
    }
}
