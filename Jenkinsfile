pipeline {
  agent any
  tools {
    maven 'Maven_3_2_5'
  }
  stages {
    stage('Compile and Run Sonar Analysis') {
      steps {
        sh '''
          mvn clean verify sonar:sonar \
          -Dsonar.projectKey=samsorbuggywebapp \
          -Dsonar.organization=samsorbuggywebapp \
          -Dsonar.host.url=https://sonarcloud.io \
          -Dsonar.token=baae1e3be609302a7c2d6368107cac510627c20a
        '''
      }
    }
    stage('Run SCA Analysis Using Snyk') {
      steps {
        script {
          withCredentials([string(credentialsId: 'SNYK_TOKEN', variable: 'SNYK_TOKEN')]) {
            sh 'mvn snyk:test -fn'
          }
        }
      }
    }
  }
}
