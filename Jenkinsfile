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
      // TODO: revert test change
      // when {
      //  branch 'main'
      //}
      stages {
        stage ('Maven Package') {
          steps {
            sh './mvnw package -DskipTests' + buildFlags
          }
        }
        stage ('Docker build') {
          steps {
            sh 'docker build -t cathychan/petclinic .'
          }
        }
        stage ('Push image') {
          steps {
            script {
              def remoteName = "cathychan/petclinic:$appVersion"
              // TODO: make sure docker credentials have been added to Jenkins
              withCredentials([usernamePassword(credentialsId: 'DH-cc-user+token', passwordVariable: 'token', usernameVariable: 'username')]) {
                sh """
                  docker login -u $username -p $token
                  docker tag cathychan/petclinic $remoteName && docker push $remoteName
                """
            }
          }
        }
      }
    }
  }
}
