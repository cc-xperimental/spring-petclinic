def buildFlags = ' -Dcheckstyle.skip'

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
        sh './mvnw clean compile' + buildFlags
      }
    }
    
    stage ('Test') {
      steps {
        sh './mvnw test' + buildFlags
      }
      
      post {
        always {
          junit 'target/**/*.xml'
        }
      }
    }
    
    stage ('Package') {
      when {
        branch 'main'
      }
      steps {
        sh './mvnw package' + buildFlags
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
