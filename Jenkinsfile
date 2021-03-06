pipeline {
  environment {
    docker_username = 'tpdockerpage'
  }
  agent any
  stages {
      stage('Clone Down') {
        options {
            skipDefaultCheckout(true)
        }
        steps {
          // node (
          //   label: 'host'
          // )
          stash (
            excludes: '.git',
            name: 'code'
          )
            
        }
      } 
      stage('Parallel execution') {
        parallel {
          stage('Say Hello') {
            steps {
              sh 'echo "hello world"'
            }
          }
          stage('Build App') {
            agent {
              docker {
                image 'gradle:jdk11'
              }
            }
            steps {
              sh 'ci/build-app.sh'
              stash 'code'
              sh 'ls'
              deleteDir()
              sh 'ls'
            }
          }
          stage('Test App'){
            agent {
              docker {
                image 'gradle:jdk11'
              }
            }
            steps {
              unstash 'code'
              sh 'ci/unit-test-app.sh'
              junit 'app/build/test-results/test/TEST-*.xml'
            }
            post {
              always {
                  deleteDir() /* clean up our workspace */
              }
            }
          }
        }
      }
        stage('push docker app'){
          options {
            skipDefaultCheckout(true)
          }
          environment {
            DOCKERCREDS = credentials('docker_login') //use the credentials just created in this stage
          }
          when {branch "master"}
          steps {
                unstash 'code' //unstash the repository code
                sh 'ci/build-docker.sh'
                sh 'echo "$DOCKERCREDS_PSW" | docker login -u "$DOCKERCREDS_USR" --password-stdin' //login to docker hub with the credentials above
                sh 'ci/push-docker.sh'
          }
        }
        stage('component test'){
          when { 
            anyOf {
              branch "master"
              changeRequest() 
              }
          }
          steps {
                unstash 'code'
                sh 'ci/component-test.sh'
          }
        }
      }  
}