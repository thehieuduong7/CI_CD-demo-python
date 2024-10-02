pipeline {
    agent any
    environment {
        DOCKER_CREDENTIALS_ID = 'docker-cred'  // need to change your Docker credentials ID
        KUBECONFIG_CREDENTIALS_ID = 'k8-cred2'  // need to change your Kubernetes credentials ID
        DOCKER_IMAGE = 'aungkyaws/python-webapp' // should be your docker hub image
    }
    stages {
        stage('Checkout') {
            steps {
                checkout scm
            }
        }
        stage('Build') {
            steps {
                script {
                    docker.build(DOCKER_IMAGE)
                }
            }
        }
        stage('Push') {
            steps {
                script {
                    docker.withRegistry('https://index.docker.io/v1/', DOCKER_CREDENTIALS_ID) {
                        docker.image(DOCKER_IMAGE).push('latest')
                    }
                }
            }
        }
        stage('Verify Workspace') {
            steps {
                sh 'ls -l'
                sh 'ls -l k8s'
            }
        }
        stage('Deploy') {
            steps {
                withKubeConfig([credentialsId: KUBECONFIG_CREDENTIALS_ID]) {
                    sh 'kubectl config view'
                    sh 'kubectl apply -f k8s/secret.yml'
                    sh 'kubectl apply -f k8s/mysql-pv.yml'
                    sh 'kubectl apply -f k8s/mysql-pvc.yml'
                    sh 'kubectl apply -f k8s/mysql-deployment.yml'
                    sh 'kubectl apply -f k8s/mysql-service.yml'
                    sh 'kubectl apply -f k8s/deployment.yml'
                    sh 'kubectl apply -f k8s/service.yml'
                    sh 'kubectl apply -f k8s/ingress.yml'
                }
            }
        }
        stage('Rollout Deployment') {
            steps {
                withKubeConfig([credentialsId: KUBECONFIG_CREDENTIALS_ID]) {
                    sh 'kubectl rollout restart deployment/python-webapp -n webapps'
                }
            }
        }
    }
    post {
        always {
            cleanWs()
        }
    }
}
