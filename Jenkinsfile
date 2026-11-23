pipeline {
  agent any
  options { timeout(time: 60, unit: 'MINUTES') }
  environment { IMAGE = "msarahm/sarah-cat"; TAG = "6" }
  stages {
    stage('Build') {
      steps {
        retry(3) { sh 'docker build -t $IMAGE:$TAG -t $IMAGE:latest .' }
      }
    }
    stage('Test') {
      steps {
        sh '''
          docker rm -f juice-test || true
          docker run -d --name juice-test $IMAGE:$TAG
          for i in $(seq 1 30); do
            docker run --rm --network container:juice-test curlimages/curl -sf http://localhost:3000 >/dev/null && exit 0
            sleep 3
          done
          echo "App did not respond"; exit 1
        '''
      }
      post { always { sh 'docker rm -f juice-test || true' } }
    }
    stage('Push to Docker Hub') {
      steps {
        withCredentials([usernamePassword(credentialsId: 'dockerhub',
            usernameVariable: 'DH_USER', passwordVariable: 'DH_TOKEN')]) {
          sh 'echo $DH_TOKEN | docker login -u $DH_USER --password-stdin'
          sh 'docker push $IMAGE:$TAG && docker push $IMAGE:latest'
        }
      }
    }
  }
}
