pipeline {
  agent any
  tools { 
    maven 'Maven_3_5_2'  
  }
  stages {
    stage('CompileandRunSonarAnalysis') {
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
  }
}
