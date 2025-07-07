pipeline {
  agent any
  tools {
    maven 'Maven_3_2_5'
  }

  environment {
    IMAGE_NAME = 'asg'
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

    stage('Build Docker Image') {
      steps {
        script {
          withDockerRegistry([credentialsId: 'dockerlogin', url: '']) {
            app = docker.build("asg")
          }
        }
      }
    }

    stage('Push Docker Image to ECR') {
      steps {
        script {
          docker.withRegistry('https://619071321115.dkr.ecr.us-west-2.amazonaws.com', 'ecr:us-west-2:aws-credentials') {
            app.push('latest')
          }
        }
      }
    }

    stage('Kubernetes Deployment of ASG Bugg Web Application') {
      steps {
        withKubeConfig([credentialsId: 'kubelogin']) {
          sh 'kubectl delete all --all -n devsecops'
          sh 'kubectl apply -f deployment.yaml --namespace=devsecops'
        }
      }
    }

    stage('wait_for_testing') {
      steps {
        sh 'pwd; sleep 180; echo "Application Has been deployed on K8S"'
      }
    }

    stage('Run DAST Using ZAP') {
      steps {
        withKubeConfig([credentialsId: 'kubelogin']) {
          sh '''
            ZAP_URL="http://$(kubectl get svc asgbuggy -n devsecops -o json | jq -r '.status.loadBalancer.ingress[0].hostname')"
            echo "Running ZAP scan on $ZAP_URL"
            zap.sh -cmd -quickurl $ZAP_URL -quickprogress -quickout ${WORKSPACE}/zap_report.html
          '''
          archiveArtifacts artifacts: 'zap_report.html'
        }
      }
    }
  }
}
