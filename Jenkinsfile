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
        container('maven') {
          sh 'mvn compile'
        }
      }
    }
    stage('Test') {
      steps {
        container('maven') {
          sh 'mvn test'
        }
      }
    }
    stage('Package') {
      parallel {
        stage('Create Jarfile') {
          steps {
            container('maven') {
              sh 'mvn package -DskipTests'
            }
          }
        }
        stage('OCI Image BnP') {
          steps { 
            container('kaniko') {
              sh "/kaniko/executor --context=`pwd` --dockerfile=`pwd`/Dockerfile --destination=docker.io/dorinatimbur/dso-demo --insecure --skip-tls-verify --cache=true"
            }
          }
        }
      }
    }
    stage('Deploy to Dev') {
      steps {
        sh "echo 'Deployment complete'"
      }
    }
  }
}
