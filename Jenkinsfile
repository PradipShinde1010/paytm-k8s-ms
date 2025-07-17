pipeline {
    agent any
    
    parameters {
        choice(
            name: 'SERVICE',
            choices: [
                'all',
                'payment-gateway',
                'wallet-service', 
                'transaction-service',
                'user-service',
                'profile-service',
                'kyc-service',
                'bank-service',
                'loan-service',
                'investment-service',
                'email-service',
                'sms-service',
                'push-service'
            ],
            description: 'Select service to deploy'
        )
        string(name: 'BUILD_NUMBER', defaultValue: '1', description: 'Build number')
        choice(name: 'ENVIRONMENT', choices: ['dev', 'staging', 'prod'], description: 'Environment')
    }
    
    environment {
        DOCKER_HUB_USERNAME = 'kastrov'
        AWS_REGION = 'us-east-1'
        EKS_CLUSTER = 'kastro-eks'
        NAMESPACE = 'paytm-app'
    }
    
    stages {
        stage('Checkout') {
            steps {
                git branch: 'main', url: 'https://github.com/KastroVKiran/paytm-k8s-ms.git'
            }
        }
        
        stage('Create Namespace') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    sh '''
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}
                        kubectl create namespace ${NAMESPACE} --dry-run=client -o yaml | kubectl apply -f -
                    '''
                }
            }
        }
        
        stage('Build and Deploy All Services') {
            when {
                expression { params.SERVICE == 'all' }
            }
            parallel {
                stage('Payment Gateway') {
                    steps {
                        script {
                            buildAndDeployService('payment-gateway', '9001')
                        }
                    }
                }
                stage('Wallet Service') {
                    steps {
                        script {
                            buildAndDeployService('wallet-service', '9002')
                        }
                    }
                }
                stage('Transaction Service') {
                    steps {
                        script {
                            buildAndDeployService('transaction-service', '9003')
                        }
                    }
                }
                stage('User Service') {
                    steps {
                        script {
                            buildAndDeployService('user-service', '9004')
                        }
                    }
                }
                stage('Profile Service') {
                    steps {
                        script {
                            buildAndDeployService('profile-service', '9005')
                        }
                    }
                }
                stage('KYC Service') {
                    steps {
                        script {
                            buildAndDeployService('kyc-service', '9006')
                        }
                    }
                }
                stage('Bank Service') {
                    steps {
                        script {
                            buildAndDeployService('bank-service', '9007')
                        }
                    }
                }
                stage('Loan Service') {
                    steps {
                        script {
                            buildAndDeployService('loan-service', '9008')
                        }
                    }
                }
                stage('Investment Service') {
                    steps {
                        script {
                            buildAndDeployService('investment-service', '9009')
                        }
                    }
                }
                stage('Email Service') {
                    steps {
                        script {
                            buildAndDeployService('email-service', '9010')
                        }
                    }
                }
                stage('SMS Service') {
                    steps {
                        script {
                            buildAndDeployService('sms-service', '9011')
                        }
                    }
                }
                stage('Push Service') {
                    steps {
                        script {
                            buildAndDeployService('push-service', '9012')
                        }
                    }
                }
            }
        }
        
        stage('Deploy Single Service') {
            when {
                expression { params.SERVICE != 'all' }
            }
            steps {
                script {
                    def portMap = [
                        'payment-gateway': '9001',
                        'wallet-service': '9002',
                        'transaction-service': '9003',
                        'user-service': '9004',
                        'profile-service': '9005',
                        'kyc-service': '9006',
                        'bank-service': '9007',
                        'loan-service': '9008',
                        'investment-service': '9009',
                        'email-service': '9010',
                        'sms-service': '9011',
                        'push-service': '9012'
                    ]
                    buildAndDeployService(params.SERVICE, portMap[params.SERVICE])
                }
            }
        }
        
        stage('Deploy Frontend') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    sh '''
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}
                        sed -i "s/BUILD_NUMBER/${BUILD_NUMBER}/g\" k8s/frontend-deployment.yaml
                        kubectl apply -f k8s/frontend-deployment.yaml -n ${NAMESPACE}
                    '''
                }
            }
        }
        
        stage('Deploy Ingress') {
            steps {
                withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
                    sh '''
                        aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}
                        kubectl apply -f k8s/ingress.yaml -n ${NAMESPACE}
                    '''
                }
            }
        }
    }
}

def buildAndDeployService(serviceName, port) {
    withCredentials([usernamePassword(credentialsId: 'dockerhub-credentials', usernameVariable: 'DOCKER_USERNAME', passwordVariable: 'DOCKER_PASSWORD')]) {
        sh """
            cd ${serviceName}
            docker build -t ${DOCKER_HUB_USERNAME}/${serviceName}:${BUILD_NUMBER} .
            docker build -t ${DOCKER_HUB_USERNAME}/${serviceName}:latest .
            echo \$DOCKER_PASSWORD | docker login -u \$DOCKER_USERNAME --password-stdin
            docker push ${DOCKER_HUB_USERNAME}/${serviceName}:${BUILD_NUMBER}
            docker push ${DOCKER_HUB_USERNAME}/${serviceName}:latest
        """
    }
    
    withCredentials([[$class: 'AmazonWebServicesCredentialsBinding', credentialsId: 'aws-credentials']]) {
        sh """
            aws eks update-kubeconfig --region ${AWS_REGION} --name ${EKS_CLUSTER}
            sed -i "s/BUILD_NUMBER/${BUILD_NUMBER}/g\" k8s/${serviceName}-deployment.yaml
            kubectl apply -f k8s/${serviceName}-deployment.yaml -n ${NAMESPACE}
        """
    }
}