pipeline {
   agent any
   environment {
       registry = "srinivas1211/k8scicd"
       GOCACHE = "/tmp"
   }
   stages {
       stage('Build') {
           agent {
               docker {
                   image 'golang'
                   args '-u root --privileged'
               }
           }
           steps {
               // Create our project directory.
               sh 'cd ${GOPATH}/src'
               sh 'mkdir -p ${GOPATH}/src/hello-world'
               // sh 'mkdir -p /.config/go/env'
               // sh 'touch /.config/go/env'
               // Copy all files in our Jenkins workspace to our project directory.
               sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
               // set GO111MODULE environment variable
               // sh 'go env -w GO111MODULE=auto'
               // set GOCACHE to off
               // sh 'go env -w GOCHACHE=off'
               // set GOENV file path
               // sh 'export GOENV="/tmp/.config/go/env"'
               // Build the app.
                  sh 'go build -buildvcs=false'
  }
      }
      stage('Test') {
          agent {
              docker {
                  image 'golang'
              }
          }
          steps {
              // Create our project directory.
              sh 'cd ${GOPATH}/src'
              sh 'mkdir -p ${GOPATH}/src/hello-world'
              // Copy all files in our Jenkins workspace to our project directory.
              sh 'cp -r ${WORKSPACE}/* ${GOPATH}/src/hello-world'
              // Remove cached test results.
              sh 'go clean -cache'
             // Run Unit Tests.
             sh 'go test ./... -v -short'
          }
      }
      stage('Publish') {
          environment {
              registryCredential = 'dockerhub'
          }
          steps{
              script {
                  def appimage = docker.build registry + ":$BUILD_NUMBER"
                  docker.withRegistry( '', registryCredential ) {
                      appimage.push()
                      appimage.push('latest')
                  }
              }
          }
      }
      stage ('Deploy') {
          steps {
              script{
                  def image_id = registry + ":$BUILD_NUMBER"
                  sh "ansible-playbook playbook.yml --extra-vars \"image_id=${image_id}\""
              }
          }
      }
   }
}
