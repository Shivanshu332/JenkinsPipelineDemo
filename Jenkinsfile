pipeline {
    agent { 
        node {
            label 'docker-terraform-aws-agent'
        }
    }
    
    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/Shivanshu332/terraformAWSdemo.git', description: 'GitHub repository URL')
        string(name: 'BRANCH', defaultValue: 'test', description: 'Branch name to checkout')
		string(name: 'AWS_Region', defaultValue: 'eu-west-1', description: 'Default AWS region')
    }

    environment {
        AWS_DEFAULT_REGION = "${params.AWS_Region}"
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    // Checking out the code from the GitHub repository
                    echo "Cloning repository ${params.REPO_URL} (branch: ${params.BRANCH})"
                    checkout([$class: 'GitSCM', branches: [[name: "*/${params.BRANCH}"]], userRemoteConfigs: [[url: params.REPO_URL]]])
                }
            }
        }

        stage('Set AWS Credentials') {
            steps {
                script {
                    // Using the AWS credentials stored in Jenkins
                    withCredentials([aws(accessKeyVariable: 'AWS_ACCESS_KEY_ID', credentialsId: 'aws-credentials', secretKeyVariable: 'AWS_SECRET_ACCESS_KEY')]) {
                        echo 'AWS Credentials are set.'
                        
                        // Explicitly export the credentials so they are available to subsequent shell commands
                        sh '''
                            # Run the AWS CLI configure commands
                            aws configure set aws_access_key_id $AWS_ACCESS_KEY_ID
                            aws configure set aws_secret_access_key $AWS_SECRET_ACCESS_KEY
                            aws configure set region $AWS_DEFAULT_REGION
                        '''
                    }
                }
            }
        }

        stage('Terraform Init') {
            steps {
                script {
                    // Run terraform init to initialize the Terraform working directory
                    echo 'Running terraform init...'
                    sh 'terraform init'
                }
            }
        }
        
        stage('Terraform Plan') {
            steps {
                script {
                    // Run terraform plan
                    echo 'Running terraform plan...'
                    sh 'terraform plan'
                }
            }
        }
    }
}
