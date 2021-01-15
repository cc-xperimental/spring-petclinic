def buildFlags = ' -Dcheckstyle.skip' // somehow checkstyle always fails, so skip

pipeline {
  agent any
  environment {
    DH_CREDS = credentials('DH-cc-user+token') // make sure this is in Jenkins
    orgName = 'cathychan' // docker hub org
    repoName = 'petclinic' // docker hub repo
    appVersion = null
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
        stash includes: '**/target/*.jar', name: 'app'
      }
    }

    stage ('Dockerize') {
      // TODO: revert test change
      // when {
      //  branch 'main'
      //}
      steps {
        unstash 'app'
        sh '''
          docker build -t ${orgName}/${repoName} .
          # tag with app version and push
          docker tag ${orgName}/${repoName} ${orgName}/${repoName}:${appVersion}
          docker push ${orgName}/${repoName}:${appVersion}
          # also push as latest
          docker push ${orgName}/${repoName}
        '''
      }
    }
  }
}
