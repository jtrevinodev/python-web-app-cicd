pipeline {
  agent any

  environment{
    registry = "jtrevinodev/python-cicd"
    registryCredential = 'docker-hub-credential'
    app = ''

    //image_tag = "${env.BRANCH_NAME}-${env.GIT_COMMIT}-${env.BUILD_NUMBER}"
    image_tag = "${env.GIT_COMMIT}-${env.BUILD_NUMBER}"
  }

  stages {

    stage('Clone repository') {

      steps { 
        script{
          checkout scm
          
        }
      }

    }

    

    stage('Build docker image') {

      steps {

        script{
          echo "Building docker container"
          app = docker.build("${registry}:${image_tag}", ".")
        }

      }

    }

    stage('Test code inside docker container'){
        steps {
            script{
                
                try {
                        app.inside() {
                            
                            echo "Runing unit test....."
                            sh "py.test --junitxml results.xml"

                            echo "Runing code coverage test....."
                            sh "coverage run -m pytest"
                            sh "coverage html"
                        }

                    } finally {
                        // Removing the docker image
                        sh "docker rmi ${image_tag}"
                    }

                
            }
        }
    }

    stage('Push tested container image to registry'){
      
      steps{

        script{
          docker.withRegistry('', registryCredential ) {
            app.push(image_tag)

          }
        }
      }
       
    }

  }

  post{
    always {
        junit 'results.xml'
    }
    success {
      mail to:"jtrevino.dev@gmail.com", subject:"SUCCESS: ${currentBuild.fullDisplayName}", body: "Build succeded."
    }
    failure {
      mail to:"jtrevino.dev@gmail.com", subject:"FAILURE: ${currentBuild.fullDisplayName}", body: "Boo, we failed."
    }
  }

}

