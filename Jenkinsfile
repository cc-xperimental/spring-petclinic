def dockerHubCredentials = 'DH-cc-user+token' // make sure this is in Jenkins
def buildFlags = ' -Dcheckstyle.skip' // somehow checkstyle always fails, so skip
def orgName = 'cathychan' // docker hub org
def repoName = 'petclinic' // docker hub repo
def appVersion = null

pipeline {
  agent any
  environment {
    DH_CREDS = credentials(dockerHubCredentials)
  }
  stages {
    stage ('Build') {
      agent {
        docker {
          image 'openjdk:8-jdk-alpine'
          args '-v /root/.m2:/root/.m2'
          reuseNode true
        }
      }
      steps {
        script {
          sh './mvnw clean compile' + buildFlags
          // retrieve app version
          // TODO: are there better ways??
          // HACK: run the command a first time to get all downloads done,
          // then run it a second time for the output
          sh './mvnw help:evaluate -Dexpression=project.version | grep "^[^\\[]"'
          appVersion = sh(script: './mvnw help:evaluate -Dexpression=project.version | grep "^[^\\[]"', returnStdout: true)
          echo "app version: $appVersion"
        }
      }
    }
    
    stage ('Test') {
      agent {
        docker {
          image 'openjdk:8-jdk-alpine'
          args '-v /root/.m2:/root/.m2'
          reuseNode true
        }
      }
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
      // TODO: revert test change
      // when {
      //  branch 'main'
      //}
      agent {
        docker {
          image 'openjdk:8-jdk-alpine'
          args '-v /root/.m2:/root/.m2'
          reuseNode true
        }
      }
      steps {
        sh './mvnw package -DskipTests' + buildFlags
      }
    }

    stage ('Dockerize') {
      // TODO: revert test change
      // when {
      //  branch 'main'
      //}
      steps {
        sh """
          docker build -t ${orgName}/${repoName} .
          docker login -u ${DH_CREDS_USR} -p ${DH_CREDS_PSW}
          docker tag ${orgName}/${repoName} ${orgName}/${repoName}:${appVersion}
          docker push ${orgName}/${repoName}:${appVersion}
          docker push ${orgName}/${repoName} // also push as latest
        """
      }
    }
  }
}
