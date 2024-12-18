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
    stage('Static Analysis') {
      parallel {
        stage('Dependency Check') {
          steps {
            container('maven') {
              catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
                sh 'mvn org.owasp:dependency-check-maven:check -Dformat=XML'
              }
            }
          }
          post {
            always {
              archiveArtifacts(allowEmptyArchive: true, artifacts: 'target/dependency-check-report.xml', fingerprint: true, onlyIfSuccessful: true)
              dependencyCheckPublisher(pattern: 'target/dependency-check-report.xml')
            }
          }
        }
        stage('OSS License Checker') {
          steps {
            container('ruby') {
              sh '''
                gem install license_finder
                license_finder
              '''
            }
          }
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
              sh '/kaniko/executor --context=`pwd` --dockerfile=`pwd`/Dockerfile --destination=docker.io/dorinatimbur/dso-demo --insecure --skip-tls-verify --cache=true'
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
