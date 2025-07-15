pipeline {
  agent any

  environment {
    REGISTRY = 'docker.io'
    IMAGE    = "tangduyenky/flask-cicd:${env.BUILD_NUMBER}"
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    /* Đăng-nhập, build & push gói gọn trong 1 stage
       xoá cấu hình cũ để chắc chắn dùng token mới mỗi lần build */
    stage('Build & Push') {
      steps {
        sh 'rm -rf $HOME/.docker || true'          // xoá cache login cũ

        withDockerRegistry(credentialsId: 'dockerhub-creds', url: '') {
          script {
            def img = docker.build("${IMAGE}")     // tự pull python:3.12-slim
            img.push()                             // đẩy image mới lên Hub
          }
        }
      }
    }

    stage('Deploy (local)') {
      steps {
        sh '''
          docker rm -f flask-cicd 2>/dev/null || true
          docker run -d --name flask-cicd -p 5000:5000 ${IMAGE}
        '''
      }
    }

        stage('Health-check') {
      steps {
        script {
          def ok = false
          5.times {
            sleep 3
            if (sh(returnStatus: true,
                   script: 'curl -sf http://localhost:5000/ > /dev/null') == 0) {
              ok = true
              echo '✅ Service is up'
              return
            }
          }
          if (!ok) { error '❌ Service did not respond with 200 OK' }
        }
      }
    }


