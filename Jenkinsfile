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
        sh './mvnw clean compile -Dcheckstyle.skip' // TODO: address checkstyle 
      }
    }
    
    stage ('Test') {
      steps {
        sh './mvnw test'
      }
      
      post {
        always {
          junit 'target/**/*.xml'
        }
      }
    }
    
    stage ('Package') {
      when {
        branch: 'main'
      }
      steps {
        sh './mvnw package'
      }
      post {
        success {
          archive 'target/**.jar'
          // TODO: replace by docker
        }
      }
    }
  }
}
