pipeline {
    agent any
    
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '935598635277.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPOSITORY = 'new/imp-repo'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
        AWS_CREDENTIALS_ID = 'aws-ecr-credentials'
        K8S_SERVER_URL = 'https://32E88A3AC7AFB373486CC102B420953C.gr7.us-east-1.eks.amazonaws.com'  // Your actual EKS API server URL
        K8S_CREDENTIALS_ID = 'k8s-service-account-token'
        HELM_RELEASE_NAME = 'petclinic-app'
        HELM_CHART_PATH = './helm/petclinic'
        K8S_NAMESPACE = 'default'
    }

    stages {
        stage('Deploy with Helm') {
            steps {
                script {
                    // Authenticate to ECR
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                        sh """
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                        """
                    }
                    
                    // Deploy using kubectl with service account token
                    withCredentials([string(credentialsId: "${K8S_CREDENTIALS_ID}", variable: 'K8S_TOKEN')]) {
                        sh """
                            # Create temporary kubeconfig
                            export KUBECONFIG=\$(mktemp)
                            kubectl config set-cluster eks-cluster --server=${K8S_SERVER_URL} --insecure-skip-tls-verify=true
                            kubectl config set-credentials jenkins-sa --token=\$K8S_TOKEN
                            kubectl config set-context jenkins-context --cluster=eks-cluster --user=jenkins-sa --namespace=${K8S_NAMESPACE}
                            kubectl config use-context jenkins-context
                            
                            # Deploy with Helm
                            /usr/local/bin/helm upgrade --install ${HELM_RELEASE_NAME} ${HELM_CHART_PATH} \
                                --namespace ${K8S_NAMESPACE} \
                                --set image.repository=${IMAGE_NAME} \
                                --set image.tag=${IMAGE_TAG} \
                                --wait
                            
                            # Verify deployment
                            kubectl get pods -n ${K8S_NAMESPACE}
                            kubectl get svc -n ${K8S_NAMESPACE}
                        """
                    }
                }
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
