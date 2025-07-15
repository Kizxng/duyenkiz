pipeline {
  agent any
  environment {
    REGISTRY = 'docker.io'
    IMAGE    = "tangduyenky/flask-cicd:${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {           // Giữ nguyên
      steps { checkout scm }
    }

    stage('Build & Push') {       // ⬅️ chỉ 1 stage duy nhất login + build + push
      steps {
        withDockerRegistry(credentialsId: 'dockerhub-creds',
                           url: 'https://index.docker.io/v1/') {
          script {
            def img = docker.build("${IMAGE}")   // pull base image OK
            img.push()                           // push tag
          }
        }
      }
    }

    stage('Deploy (local)') {     // chạy container mới
      steps {
        sh '''
          docker rm -f flask-cicd 2>/dev/null || true
          docker run -d --name flask-cicd -p 5000:5000 ${IMAGE}
        '''
      }
    }

    stage('Health-check') {       // kiểm tra
      steps {
        sh 'curl -f http://localhost:5000/'
      }
    }
  }
}

