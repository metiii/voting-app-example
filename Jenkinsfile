pipeline {
  agent none

  stages{
      stage("worker-build"){
        agent{
          docker{
          image 'maven:3.6.1-jdk-8-slim'
          args '-v $HOME/.m2:/root/.m2'
          }
        }

        when{
          changeset "**/worker/**"
        }
        steps{
            echo 'Compiling worker app'
            dir('worker'){
            sh 'mvn compile'
            }
        }
      }

      stage("worker-test"){
        agent{
          docker{
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
            }
         }

        when{
          changeset "**/worker/**"
          }
        steps{
            echo 'Running Unit Tests on worker app'
            dir('worker'){
              sh 'mvn clean test'
            }

            }
        }
      stage("worker-package"){
        agent{
          docker{
            image 'maven:3.6.1-jdk-8-slim'
            args '-v $HOME/.m2:/root/.m2'
            }
          }
        when{
          changeset "**/worker/**"
        }

        steps{
            echo 'Packaging worker app'
            dir('worker'){
            sh 'mvn package -DskipTests'
            archiveArtifacts artifacts: '**/target/*.jar', fingerprint: true
            }
        }
        }
     stage('docker-package'){
       agent any
       when{
          changeset "**/worker/**"
       }
       steps{
         echo 'Packing worker app with docker'
         script{
           docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
             def workerImage = docker.build("metiii/worker:v${env.BUILD_ID}", "./worker")
             workerImage.push()
             workerImage.push("latest")
          }
         }
        }
       }

    stage("vote-build"){
        agent{
          docker{
            image 'python:2.7.16-slim'
             args '--user root'
          }
        }
        when{
          changeset "**/vote/**"
        }
        steps{
            echo 'Compiling worker app'
            dir('vote'){
            sh 'pip install -r requirements.txt'
            }
        }
      }
    stage("vote-test"){
        agent{
           docker{
             image 'python:2.7.16-slim'
             args '--user root'
          }
        }
        when{
          changeset "**/vote/**"
          }
        steps{
            echo 'Running Unit Tests on worker app'
            dir('vote'){
              sh 'pip install -r requirements.txt'
              sh 'nosetests -v'

            }

            }
        }
    stage('vote-docker-package'){
       agent any
       when{
          changeset "**/vote/**"
       }
       steps{
         echo 'Packing vote app with docker'
         script{
           docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
             def workerImage = docker.build("metiii/vote:v${env.BUILD_ID}", "./vote")
             workerImage.push()
             workerImage.push("latest")
          }
         }
        }
       }

    
    stage("result-build"){
      agent{
        docker{
          image 'node:lts-buster-slim'
        }
      }
        when{
      changeset "**/result/**"
    }
      steps{
        echo 'Compiling result app'
        dir('result'){
          sh 'npm install'
        }
        }
      }

    stage("result-test"){
        agent{
          docker{
            image 'node:lts-buster-slim'
            }
         }

        when{
          changeset "**/result/**"
          }
        steps{
          echo 'Running Unit Tests on result app'
          dir('result'){
              sh 'npm install'
              sh 'npm test'
            }

            }
        }
    stage('result-docker-package'){
      agent any
        when{
          changeset "**/result/**"
       }
      steps{
        echo 'Packing result app with docker'
        script{
        docker.withRegistry('https://index.docker.io/v1/', 'dockerlogin') {
          def resultImage = docker.build("metiii/result:latest${env.BUILD_ID}", "./result")
          resultImage.push()
          resultImage.push("latest")
          }
        }
        }
    }

    stage('deploy to dev'){
      agent any
      when{
        branch 'master'
        }
      steps{
        echo 'Deploy instavote app with docker compose'
        sh 'docker-compose up -d'
        }
    }


  post{
    always{
      echo 'Building multibranch pipeline for worker is completed..'
    }
    failure{
    slackSend (channel: "instavote-cd", message: "Build Failed - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }
    success{
      slackSend (channel: "instavote-cd", message: "Build Succeeded - ${env.JOB_NAME} ${env.BUILD_NUMBER} (<${env.BUILD_URL}|Open>)")
    }
  }
}
}

