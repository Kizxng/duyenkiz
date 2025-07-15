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

    stage('Build & Push') {
      steps {
        sh 'rm -rf $HOME/.docker || true'          // xoá cache login cũ

        withDockerRegistry(credentialsId: 'dockerhub-creds', url: '') {
          script {
            def img = docker.build("${IMAGE}")     // pull + build
            img.push()                             // push lên Hub
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
            sleep 3                                 // đợi 3 s
            if (sh(returnStatus: true,
                   script: 'curl -sf http://localhost:5000/ > /dev/null') == 0) {
              ok = true
              echo '✅ Service is up'
              return
            }
          }
          if (!ok) {
            error '❌ Service did not respond with 200 OK in 15 s'
          }
        }
      }
    }
  }
}

