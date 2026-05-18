pipeline {
    agent any

    environment {
        REGISTRY = "docker.io/YOUR_USERNAME"
        IMAGE_TAG = "${BUILD_NUMBER}"
        NAMESPACE = "petclinic"
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Maven') {
            steps {
                sh 'mvn clean package -DskipTests'
            }
        }

        stage('Build Images') {
            steps {
                sh '''
                    docker build -t $REGISTRY/api-gateway:$IMAGE_TAG api-gateway
                    docker build -t $REGISTRY/config-server:$IMAGE_TAG config-server
                    docker build -t $REGISTRY/customers-service:$IMAGE_TAG customers-service
                    docker build -t $REGISTRY/discovery-server:$IMAGE_TAG discovery-server
                '''
            }
        }

        stage('Deploy with Helm') {
            steps {
                sh '''
                    oc project $NAMESPACE

                    helm upgrade --install petclinic ./helm/petclinic \
                      -n $NAMESPACE \
                      --set services.api-gateway.image=$REGISTRY/api-gateway:$IMAGE_TAG \
                      --set services.config-server.image=$REGISTRY/config-server:$IMAGE_TAG \
                      --set services.customers-service.image=$REGISTRY/customers-service:$IMAGE_TAG \
                      --set services.discovery-server.image=$REGISTRY/discovery-server:$IMAGE_TAG
                '''
            }
        }
    }

    post {
        success {
            echo "Pipeline Success"
        }
        failure {
            echo "Pipeline Failed"
        }
    }
}
