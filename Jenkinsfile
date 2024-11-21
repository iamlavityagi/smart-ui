pipeline {
  environment {
    dockerimagename = "iamlavityagi/smart-ui:latest"
    dockerImage = ""
  }
  agent any
  stages {
    stage('Checkout Source') {
      steps {
		script {
            echo 'Checking out source code...'
            git 'https://github.com/iamlavityagi/smart-ui.git'
        }
      }
    }
    stage('Build image') {
      steps{
        script {
		  echo 'Building Docker image...'	
          dockerImage = docker.build dockerimagename
        }
      }
    }
    stage('Pushing Image') {
      environment {
          registryCredential = 'docker'
           }
      steps{
        script {
		  echo 'Pushing Docker image to Docker Hub...'
          docker.withRegistry( 'https://registry.hub.docker.com', registryCredential ) {
            dockerImage.push("latest")
          }
        }
      }
    }
    stage('Deploying React.js container to Kubernetes') {
      steps {
        script {
          script {
			echo 'Deploying to Kubernetes...'
                    // Extract the deployment name dynamically from the YAML file
                    def deploymentName = sh(script: "grep 'name:' deployment.yaml | awk '{print \$2}' | head -1", returnStdout: true).trim()
                    
                    // Apply the deployment YAML
                    sh 'kubectl apply -f deployment.yaml'
                    sh 'kubectl apply -f service.yaml'
                    
                    // Force the deployment restart dynamically
                    echo "Restarting deployment: ${deploymentName}"
                    sh "kubectl rollout restart deployment/${deploymentName}"
			}
        }
      }
    }
  }
  post {
        always {
            script {
                echo 'Cleaning up local Docker image...'
                sh "docker rmi ${dockerimagename} || true"
            }
        }
        success {
            echo 'Pipeline executed successfully!'
        }
        failure {
            echo 'Pipeline failed. Please check the logs.'
        }
    }
}