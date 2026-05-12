pipeline {
    agent any
    environment {
        IMAGE_NAME = "python-flask-app"
        DOCKER_HUB = credentials('docker-hub-creds')
        // Change GITHUB_USERNAME & GITHUB_REPOSITORY_NAME values accordingly
        GITHUB_USERNAME="mnakib"
        GITHUB_REPOSITORY_NAME="jenkins-demo"
        OCP_API = "https://api.ocp4.example.com:6443"
        OCP_USER = "admin"
        OCP_PASS = "redhatocp"
        IMAGE_PATH = "docker.io/mouradn81/python-flask-app:latest"
    }
    stages {
        stage('Checkout Source') {
            steps {
                // Pull the code from your repository
                git branch: 'main', url: "https://github.com/${GITHUB_USERNAME}/${GITHUB_REPOSITORY_NAME}.git"
            }
        }
        stage('Build & Test') {
            steps {
                // Instead of docker.inside, we run a container manually
                sh '''
                    docker run --rm -v $(pwd):/app -w /app python:3.9-slim bash -c "
                        pip install flask pytest && 
                        pytest
                    "
                '''
            }
        }
        stage('Build & Push') {
            steps {
                sh "docker build -t ${DOCKER_HUB_USR}/${IMAGE_NAME}:latest ."
                sh "echo ${DOCKER_HUB_PSW} | docker login -u ${DOCKER_HUB_USR} --password-stdin"
                sh "docker push ${DOCKER_HUB_USR}/${IMAGE_NAME}:latest"
            }
        }
        stage('Deploy to OpenShift') {
            steps {
                script {
                    // Define your target namespace and deployment name
                    def NAMESPACE_NAME = "default"
                    def DEPLOYMENT_NAME = "python-flask-app"
                    // Login, create the namespace if doesn't exist then swith to it
                    // create the app if it doesn't exist, or update the image if it does
                    sh """
                    oc login ${OCP_API} -u ${OCP_USER} -p ${OCP_PASS} --insecure-skip-tls-verify   
                    oc new-project ${NAMESPACE_NAME} || echo "Namespace already exists"
                    oc project ${NAMESPACE_NAME}
                    oc create deployment ${DEPLOYMENT_NAME} --image ${IMAGE_PATH} --namespace=${NAMESPACE_NAME} || oc patch deployment/${IMAGE_NAME} -p '{"spec":{"template":{"spec":{"containers":[{"name":"${IMAGE_NAME}","image":"${IMAGE_PATH}"}]}}}}'
                    # Expose the deployment
                    oc expose deployment ${DEPLOYMENT_NAME} --target-port 5000 --port 80 --namespace=${NAMESPACE_NAME}
                    # Expose the deployment
                    oc expose svc/${IMAGE_NAME} --namespace=${NAMESPACE_NAME} || echo "Route already exists" 
                    """
                }
            }
        }
    }
}
