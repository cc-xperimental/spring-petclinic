def buildFlags = ' -Dcheckstyle.skip'

pipeline {
  agent any
  stages {
    stage ('Build and Test') {
      agent {
        docker {
          image 'openjdk:8-jdk-alpine'
          args '-v /root/.m2:/root/.m2'
        }
      }
      stages {
        stage ('Maven Build') {
          steps {
            sh './mvnw clean compile' + buildFlags
          }
        }
        
        stage ('Maven Test') {
          steps {
            sh './mvnw test' + buildFlags
          }          
          post {
            always {
              junit 'target/**/*.xml'
            }
          }
        }
      }
    }
    
    stage ('Dockerize') {
      when {
        branch 'main'
      }
      stages {
        stage ('Maven Package') {
          steps {
            sh './mvnw package -DskipTests' + buildFlags
          }
        }
        stage ('Docker image') {
          steps {
            sh 'docker build -t cctest/petclinic .'
          }
        }
      }
    }
  }
}
