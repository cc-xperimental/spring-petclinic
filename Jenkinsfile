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
    
    stage ('Package for master') {
      when {
        branch 'master'
      }
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

    stage ('Dockerize for master') {
      when {
        branch 'master'
      }
      steps {
        unstash 'app'
        sh '''
          docker login -u ${DH_CREDS_USR} -p ${DH_CREDS_PSW}
          docker build --force-rm -t ${orgName}/${repoName} .
          # tag with app version and push
          docker tag ${orgName}/${repoName} ${orgName}/${repoName}:${appVersion}
          docker push ${orgName}/${repoName}:${appVersion}
          # also push as latest
          docker push ${orgName}/${repoName}
        '''
      }
    }
  }
  post {
    always {
      script {
        if (env.BRANCH_NAME = 'master') {
          def status = sh(script: 'docker rmi --force ${orgName}/${repoName} ${orgName}/${repoName}:${appVersion}', returnStatus: true) // ignore failure
        }
        cleanWs()
      }
    }
  }
}
