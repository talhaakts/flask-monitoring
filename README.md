# Flask-App
git clone https://github.com/hbayraktar/flask-monitoring.git
cd flask-monitoring
bash jenkins.sh
-----
jenkins.sh içeriği

#!/bin/bash
sudo apt update -y
sudo apt install openjdk-17-jdk -y
sudo wget -O /usr/share/keyrings/jenkins-keyring.asc \
https://pkg.jenkins.io/debian/jenkins.io-2023.key
echo "deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc]" \
https://pkg.jenkins.io/debian binary/ | sudo tee \
/etc/apt/sources.list.d/jenkins.list > /dev/null
sudo apt-get update -y
sudo apt-get install jenkins -y
# Add Docker's official GPG key:
sudo apt-get update -y
sudo apt-get install ca-certificates curl -y
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
sudo systemctl enable docker

-----
curl ifconfig.co
167.172.99.134
http://167.172.99.134:8080/
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

Jenkins'in Docker ile entegre çalışması için bazı eklentiler gereklidir. Bu eklentileri Jenkins arayüzü üzerinden yükleyebilirsiniz:

Gerekli Pluginler:
    Docker Pipeline Plugin
    Git Plugin
    Pipeline Plugin
    Kubernetes Plugin:
    Credentials Plugin (Docker Hub gibi hizmetler için kimlik bilgileri yönetimi)

Jenkins Pipeline'ı Çalıştırma
    
1-Jenkins Web Arayüzünde Yeni Bir Job Oluşturun:

"New Item" butonuna tıklayın.
Proje adını flask-monitoring olarak belirleyin.
"Pipeline" türünü seçin ve "OK" butonuna tıklayın.
Pipeline Yapılandırması:

Pipeline sekmesine gidin.
Definition bölümünde "Pipeline script from SCM" seçeneğini seçin.
SCM olarak "Git" seçin.
Repository URL kısmına https://github.com/hakanbayraktar/flask-monitoring.git yazın.
"Save" butonuna tıklayın.

DockerHub ve kubernetes sertifika ve Kimlik Bilgileri:
Jenkins yönetim arayüzünde "Credentials" bölümüne gidin.
    DockerHub kullanıcı adı ve şifresini ID olarak dockerhub-cred ekleyin.
    Kind: Kubernetes configuration (kubeconfig) seçin.
Kubeconfig dosyanızı yükleyin ya da manuel olarak gerekli bilgileri girin.
Credential ID'sini belirleyin (k8s-cluster-cred).

Pipeline'ı Çalıştırma:

Pipeline'ı başlatmak için "Build Now" butonuna tıklayın.

Dockerfile---
FROM python:3.11-slim

WORKDIR /app

COPY requirements.txt .

RUN pip install --no-cache-dir -r requirements.txt

COPY . .

ENV FLASK_RUN_HOST=0.0.0.0

EXPOSE 5000

CMD [ "flask", "run"]

app.py---
import psutil
from flask import Flask, render_template

app = Flask(__name__)

@app.route("/")
def index():
    cpu_metric = psutil.cpu_percent()
    mem_metric = psutil.virtual_memory().percent
    Message = None
    if cpu_metric > 70 or mem_metric > 70:
        Message = "High CPU or Memory Detected, scale up!!!"
    return render_template("index.html", cpu_metric=cpu_metric, mem_metric=mem_metric, message=Message)

if __name__=='__main__':
    app.run(debug=True, host = '0.0.0.0')

 jenkinsfile----
pipeline {
    agent any
    }

    stages {
        stage('git cloning') {
            steps {
                echo 'cloning git repo'
                git url:"https://github.com/hbayraktar/flask-monitoring.git", branch: "main"
            }
        }
        stage('build') {
            steps {
                echo 'building the image'
                sh "docker build -t flask-monitoring ."
                
            }
        }
        stage('dockerhub') {
            steps {
                echo 'pushing to dockerhub'
                withCredentials([usernamePassword(credentialsId:"dockerhub-cred",usernameVariable:"user",passwordVariable:"pass")]){
                sh "docker login -u ${env.user} -p ${env.Pass}"
                sh "docker tag monitoring-flask-app ${env.user}/monitoring-flask-app:latest"
                sh "docker push  ${env.user}/flask-monitoring:latest"
                }
            }
        }
        stage('kubernetes') {
            steps {
               
                sh "kubectl apply -f deployment.yaml"
                sh "kubectl apply -f service.yaml"
            }
        }
    }
}
    # flask-monitoring
