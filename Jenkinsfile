pipeline {
  agent {
    docker {
      image 'maven:alpine'
      args '-v /var/run/docker.sock:/var/run/docker.sock -v $HOME/.m2:/root/.m2'
    }
  }
  parameters {
    booleanParam(name: 'DEPLOY_TO_REP', defaultValue: false, description: 'Tick this box to deploy build artifact to release environment')
  }
  options {
    buildDiscarder(logRotator(numToKeepStr: '5'))
    disableConcurrentBuilds()
  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn clean install -DskipTests=true -f $PWD/sonarqube-scanner-maven/pom.xml'
      }
    }
    stage('Test') {
      failFast true
      parallel {
        stage('Unit tests') {
          steps {
            sh 'mvn test -f $PWD/sonarqube-scanner-maven/pom.xml'
          }
        }
        stage('Integration tests') {
          steps {
            echo 'Preparing environment'
            echo 'Deploying application'
            echo 'Running integration tests'
          }
        }
      }
    }
    stage('Code Quality') {
      steps {
        sh 'mvn org.sonarsource.scanner.maven:sonar-maven-plugin:3.4.0.905:sonar -f $PWD/sonarqube-scanner-maven/pom.xml -Dsonar.host.url=http://node1:9000'
      }
    }
    stage('REL deployment') {
      when {
        expression { return params.DEPLOY_TO_REP }
      }
      steps {
        echo 'Deploying to Release environment'
      }
    }
    stage('UAT deployment') {
      when {
	     branch 'master'
      }
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          input 'Should I deploy UAT?'
	        echo 'Deploying to UAT'
        }
      }
    }
    stage('Prod deployment') {
      when {
	     branch 'master'
      }
      steps {
        timeout(time: 5, unit: 'MINUTES') {
          input 'Should I deploy to PROD?'
	        echo 'Deploying to production'
        }
      }
    }
  }
  post {
    always {
      archive 'target/**/*'
      junit(testResults: '**/target/surefire-reports/*.xml', allowEmptyResults: true)
      archiveArtifacts(artifacts: '**/target/*.jar', fingerprint: true, onlyIfSuccessful: true, defaultExcludes: true)
    }
  }
}
