pipeline {
  agent {
    docker {
      image 'openjdk:8-jdk-alpine'
      args '-v /root/.m2:/root/.m2'
    }
  }
  
  stages {
    stage ('Build') {
      steps {
        sh './mvnw clean install'
      }
    }
  }
}
