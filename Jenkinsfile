pipeline {
    agent any
    
    environment {
        KUBE_CONFIG = credentials('k8s-config')
    }

    stages {
        stage('Build') {
            steps {
                sh '''
                echo 'Building App...'
                docker build -t flask-app:latest .
                '''
                }
            }
        
        stage('Pushing to ECR') {
            steps {
                sh '''
                aws ecr-public get-login-password --region us-east-1 | docker login --username AWS --password-stdin public.ecr.aws/k1j1o3o8
                docker build -t anatoly/rs-task .
                docker tag anatoly/rs-task:latest public.ecr.aws/k1j1o3o8/anatoly/rs-task:latest
                docker push public.ecr.aws/k1j1o3o8/anatoly/rs-task:latest
                '''
            }
        }
        stage('Deploying app to K8s cluster') {
            steps {
                withCredentials([file(credentialsId: 'k8s-config', variable: 'KUBECONFIG')]) {
                    sh '''
                    helm upgrade --install flask-app ./helm-chart/ \
                    --set image.repository=public.ecr.aws/k1j1o3o8/anatoly/rs-task \
                    --set image.tag=latest
                    '''
                
            }
        }
    }
    post {
        always {
            echo 'Pipeline completed.'
        }
        success {
            echo 'Pipeline executed successfully.'
        }
        failure {
            echo 'Pipeline failed.'
        }
    }
}
