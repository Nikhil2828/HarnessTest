pipeline {
    agent any

    environment {
        DOCKERHUB_USER = "nganta0102"
        IMAGE_NAME = "cicd-demo"
        IMAGE_TAG = "${BUILD_NUMBER}"
        RELEASE_NAME = "cicd-demo"
        CHART_PATH = "./cicd-demo-chart"
    }

    stages {
        
	stage('Debug Workspace') {
 	    steps {
                sh """
                pwd
                ls -la
                """
    	    }
        }
	stage('Build Image') {
            steps {
                sh """
                docker build -t $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG .
                docker tag $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG $DOCKERHUB_USER/$IMAGE_NAME:latest
                """
            }
        }

        stage('Push Image') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'dockerhub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh """
                    echo $DOCKER_PASS | docker login -u $DOCKER_USER --password-stdin
                    docker push $DOCKERHUB_USER/$IMAGE_NAME:$IMAGE_TAG
                    docker push $DOCKERHUB_USER/$IMAGE_NAME:latest
                    """
                }
            }
        }

        stage('Deploy Dev') {
            steps {
                sh """
                helm upgrade --install $RELEASE_NAME-dev $CHART_PATH \
                  -f $CHART_PATH/values-dev.yaml \
                  --set image.repository=$DOCKERHUB_USER/$IMAGE_NAME \
                  --set image.tag=$IMAGE_TAG
                """
            }
        }
        stage('Deploy QA') {
            steps {
                sh """
                helm upgrade --install $RELEASE_NAME-dev $CHART_PATH \
                  -f $CHART_PATH/values-qa.yaml \
                  --set image.repository=$DOCKERHUB_USER/$IMAGE_NAME \
                  --set image.tag=$IMAGE_TAG
                """
            }
        }

	stage('Deploy Prod') {
            steps {
                sh """
                helm upgrade --install $RELEASE_NAME-dev $CHART_PATH \
                  -f $CHART_PATH/values-prod.yaml \
                  --set image.repository=$DOCKERHUB_USER/$IMAGE_NAME \
                  --set image.tag=$IMAGE_TAG
                """
            }
        }

        stage('Verify') {
            steps {
                sh """
                kubectl get pods
                kubectl get svc
                """
            }
        }
    }
}
