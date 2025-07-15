pipeline {
  agent any

  environment {
  REGISTRY = "docker.io"
  IMAGE    = "tangduyenky/flask-cicd:${env.BUILD_NUMBER}"
}
  stages {
    stage('Checkout') { steps { checkout scm } }

    stage('Build image') {
      steps { script { docker.build("${IMAGE}") } }
    }

  stage('Push image') {
  steps {
    withCredentials([usernamePassword(credentialsId: 'dockerhub-creds',
                                      usernameVariable: 'DU',
                                      passwordVariable: 'DP')]) {
      sh """
        echo "$DP" | docker login -u "$DU" --password-stdin
        docker push ${IMAGE}
        docker logout
      """
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
        sh 'sleep 5'
        sh 'curl -f http://localhost:5000/'
      }
    }
  }

  post { success { echo '🎉 Deploy OK!' } }
}
