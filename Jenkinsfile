pipeline {
    agent any
    
    environment {
        AWS_REGION = 'us-east-1'
        ECR_REGISTRY = '935598635277.dkr.ecr.us-east-1.amazonaws.com'
        ECR_REPOSITORY = 'new/imp-repo'
        IMAGE_TAG = "${env.BUILD_NUMBER}"
        IMAGE_NAME = "${ECR_REGISTRY}/${ECR_REPOSITORY}"
        AWS_CREDENTIALS_ID = 'aws-ecr-credentials'
        HELM_RELEASE_NAME = 'petclinic-app'
        HELM_CHART_PATH = './helm/petclinic'
        K8S_NAMESPACE = 'default'
    }

    stages {
        stage('Deploy with Helm') {
            steps {
                script {
                    // Use AWS credentials for both ECR and EKS access
                    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: "${AWS_CREDENTIALS_ID}"]]) {
                        sh """
                            # ECR login
                            aws ecr get-login-password --region ${AWS_REGION} | docker login --username AWS --password-stdin ${ECR_REGISTRY}
                            
                            # Set up kubeconfig using AWS CLI (same as your management host)
                            export KUBECONFIG=/var/lib/jenkins/.kube/config
                            
                            # Update kubeconfig for EKS
                            aws eks update-kubeconfig --region ${AWS_REGION} --name my-eks-cluster-5101
                            
                            # Test connectivity
                            kubectl get nodes
                            
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
