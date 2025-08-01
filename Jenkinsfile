pipeline {
    agent any

    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '935598635277.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPOSITORY = 'new/imp-repo'
        IMAGE_TAG = "${env.BUILD_NUMBER}"   // Or set as param
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
        AWS_CREDENTIALS_ID = 'aws-ecr-credentials'  // Jenkins AWS Credentials ID
        KUBE_CONFIG_CREDENTIALS_ID = 'kubeconfig-credentials'  // Jenkins Credentials ID for kubeconfig
        HELM_RELEASE_NAME = 'petclinic-app'
        HELM_CHART_PATH = './helm/petclinic'  // Path to your Helm chart
        K8S_NAMESPACE = 'default'  // Or a custom namespace
    }

    stages {
        stage('Set Up Kubeconfig') {
            steps {
                withCredentials([file(credentialsId: "${KUBE_CONFIG_CREDENTIALS_ID}", variable: 'KUBECONFIG')]) {
                    sh 'export KUBECONFIG=$KUBECONFIG && kubectl version'
                }
            }
        }

        stage('Deploy with Helm') {
            steps {
                script {
                    // Log in to ECR (using AWS CLI) to ensure cluster can pull image
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                        sh """
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                        """
                    }
                    // Deploy using Helm
                    sh """
                        /usr/local/bin/helm upgrade --install ${HELM_RELEASE_NAME} ${HELM_CHART_PATH} \
                        --namespace ${K8S_NAMESPACE} \
                        --set image.repository=${IMAGE_NAME} \
                        --set image.tag=${IMAGE_TAG} \
                        --wait
                    """
                }
            }
        }

        stage('Verify Deployment') {
            steps {
                sh "kubectl get pods -n ${K8S_NAMESPACE}"
                sh "kubectl get svc -n ${K8S_NAMESPACE}"
            }
        }
    }

    post {
        success {
            echo "Deployment succeeded: ${HELM_RELEASE_NAME} updated with image tag ${IMAGE_TAG}"
        }
        failure {
            echo "Deployment failed. Check logs for errors."
        }
    }
}
