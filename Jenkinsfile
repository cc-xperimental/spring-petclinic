def buildFlags = ' -Dcheckstyle.skip'
def appVersion = null

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
            script {
              // retrieve app version
              appVersion = sh(script: './mvnw help:evaluate -Dexpression=project.version | grep "^[^\\[]"', returnStdout: true)
              echo "app version: $appVersion"
              sh './mvnw clean compile' + buildFlags
            }
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
            script {
              sh './mvnw package -DskipTests' + buildFlags
              // retrieve app version
              appVersion = sh(script: './mvnw help:evaluate -Dexpression=project.version | grep "^[^\\[]"', returnStdout: true)
              echo "app version: $appVersion"
            }
          }
        }
        stage ('Docker build') {
          steps {
            sh 'docker build -t cctest/petclinic .'
          }
        }
        stage ('Push image') {
          steps {
            echo "TBA"
          }
        }
      }
    }
  }
}
