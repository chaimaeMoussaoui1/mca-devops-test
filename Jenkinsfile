pipeline {
    agent {
    kubernetes {
      inheritFrom  'buildah'
    }
    }
 

    stages {

        // stage('Checkout') {
        //     steps {
        //         git ''
        //     }
        // }

        stage('Build backend image') {
            steps {
              container('buildah'){ 
                sh 'buildah build --tag backend:ci ./backend'
            }
            }
        }
        stage('Build frontend image') {
            steps {
              container('buildah'){ 
                sh 'buildah build --tag frontend:ci ./frontend'
            }
            }
        }
    }

    post {
        always {
            echo 'Pipeline Finished'
        }
        success {
            echo 'Deployment successful'
        }
        failure {
            echo 'Pipeline failed'
        }
    }
}

