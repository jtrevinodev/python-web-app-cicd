pipeline {
  agent any

  environment{
    registry = "jtrevinodev/python-cicd"
    registryCredential = 'docker-hub-credential'
    app = ''

    image_tag = "${env.BRANCH_NAME}-${env.GIT_COMMIT}-${env.BUILD_NUMBER}"
  }

  stages {

    stage('Clone repository') {

      /*steps {
        git branch: 'master', credentialsId: 'github-key', url: 'git@github.com:jtrevinodev/guestbook-devops.git'

      }*/

      steps { 
        script{
          checkout scm
          //git credentialsId: 'github-key', url: 'git@github.com:jtrevinodev/guestbook-devops.git'
        }
      }

    }

    

    stage('Build docker image') {

      steps {

        script{
          echo "Building docker container"
          app = docker.build("${registry}", ".")
        }

      }

    }

    stage('Test code inside docker container'){
        steps {
            script{
                //app.run()
                app.withRun('-d=true -p 8888:8080') {c ->
                    c.inside{
                        /*  Do something here inside container  */
                        sh "ls"
                        sh "python -m unittest --verbose --failfast"
                    }
                }
                /*app.inside() {
                    sh "python -m unittest --verbose --failfast"
                }*/
            }
        }
    }

    stage('Push container image to registry'){
      
      steps{

        script{
          docker.withRegistry('', registryCredential ) {
            app.push(image_tag)

          }
        }

        
        
      }
       
    }
    

    /*stage('Deploy to Kubernetes') {
      steps{
        script{
          sh('echo "Deploying to production environment"')
          
          sh 'echo "Clonning deployment repository"'

          //docker.image('argoproj/argo-cd-ci-builder:v1.0.0').inside {
          dir("deploy") {
            git credentialsId: 'github-key', url: 'git@github.com:jtrevinodev/guestbook-devops-deploy.git'

            def frontend_df = "base/resources/frontend-deployment.yaml"
            def frontend_deployment = readFile frontend_df
            frontend_deployment = frontend_deployment.replaceAll("image:.*", "image: jtrevinodev/guestbook:${image_tag}")
            writeFile file: frontend_df, text: frontend_deployment
            sh("cat ${frontend_df}")

            sh 'echo "Pushing deployment config to deployment repository"'
            
            //withCredentials([GitUsernamePassword(credentialsId: 'github-accesstoken', gitToolName: 'Default')]){
            //withCredentials([sshUserPrivateKey(credentialsId: "github-key", keyFileVariable: 'key')]) {
            withCredentials([string(credentialsId: 'github-jenkins-ak', variable: 'TOKEN')]) {
            // For SSH private key authentication, try the sshagent step from the SSH Agent plugin.
            //  sshagent (credentials: ['github-key']) {
              sh 'git config --global user.email "jtrevino.dev@gmail.com"'
              sh 'git config --global user.name "Jenkins pipeline"'
              //sh 'git checkout master'
              sh "git add ${frontend_df}"
              sh 'git commit -m "image tag updated: ${image_tag}"'
              
              sh 'git remote set-url origin https://$TOKEN@github.com/jtrevinodev/guestbook-devops-deploy.git'

              sh 'git push origin master'
            }

          }
        }
        
      }
      
      

    }*/

  }
  post{
    success {
      mail to:"jtrevino.dev@gmail.com", subject:"SUCCESS: ${currentBuild.fullDisplayName}", body: "Build succeded."
    }
    failure {
      mail to:"jtrevino.dev@gmail.com", subject:"FAILURE: ${currentBuild.fullDisplayName}", body: "Boo, we failed."
    }
  }

}

