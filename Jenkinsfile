def builderDocker
def CommitHash

pipeline {
    agent any

    parameters {
       booleanParam(name:'RUNTEST',defaultValue: true, description: 'Toggle this value for testing')
       choice(name:'CICD',choices: ['CI', 'CICD'], description: 'Pick something')       
    } 

  stages {
    stage('Build Project') {
      steps {
            nodejs("node12") {
            sh 'npm install'
            }
      }
    }

   stage('Build Docker Images') {

       steps{
           script {
                CommitHash = sh (script : "git log -n 1 --pretty=format:'%H'", returnStdout:true)
                builderDocker = docker.build("frontend:${CommitHash}")
           }
       }
   }

   stage('Run Testing') {
        when {
            expression {
                params.RUNTEST
            }
        }
        steps {
            script {
                builderDocker.inside {
                    sh 'echo passed'
                }
            }
        }
   }

   stage('Deploy') {
       when {
           expression {
               params.CICD == 'CICD'
           }
       }
       
       steps {
         echo 'Deploy...'
       }
   }

  }
}
