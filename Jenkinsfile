pipeline {
  agent {
    kubernetes {
      yamlFile 'build-agent.yaml'
      defaultContainer 'maven'
      idleMinutes 1
    }
  }
  stages {
    stage('Build') {
      steps {
        sh 'mvn compile'
      }
    }
    stage('Test') {
      steps {
        sh 'mvn test'
      }
    }
    stage('Package') {
      steps {
        sh 'mvn package -DskipTests'
      }
    }
    stage('OSS License Compliance Check') {
      steps {
        sh 'mvn license:check'
      }
    }
    stage('SCA') {
      steps {
        sh 'mvn org.owasp:dependency-check-maven:check'
      }
      post {
        always {
          archiveArtifacts allowEmptyArchive: true, artifacts: 'target/dependency-check-report.html', fingerprint: true, onlyIfSuccessful: true
        }
      }
    }
    stage('Deploy to Dev') {
      steps {
        sh "echo 'Deployment done'"
      }
    }
  }
}
