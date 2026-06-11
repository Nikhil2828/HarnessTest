pipeline {
    agent any
   
    environment {
        IMAGE_NAME = "cicd-demo"
        IMAGE_TAG = "1.0.${BUILD_NUMBER}"
        RELEASE_NAME = "cicd-demo"
        CHART_PATH = "./cicd-demo-chart"
    }

    stages {
        stage('Checkout') {
            steps {
                echo 'Checking out source code'
            }
        }

        stage('Build Docker Image') {
            steps {
                sh """
                docker build -t ${IMAGE_NAME}:${IMAGE_TAG} .
		docker tag ${IMAGE_NAME}:${IMAGE_TAG} ${IMAGE_NAME}:latest
		"""
	     }
        }

	stage('Helm Deploy to Kubernetes') {
	    steps {
		sh """
		helm upgrade --install ${RELEASE_NAME} ${CHART_PATH} \
		  --set image.repository=${IMAGE_NAME} \
                  --set image.tag=latest \
		  --set image.pullPolicy=IfNotPresent
		"""
            }
	}

	stage('Verify Deployment') {
	   steps { 
	       sh """
	       kubectl get pods
	       kubectl get svc
	       """
           }
 	}
     }
  }

