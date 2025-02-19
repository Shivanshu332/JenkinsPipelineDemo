pipeline {
    agent { 
        node {
            label 'docker-terraform-aws-agent'
        }
    }

    triggers {
        pollSCM('H/5 * * * *')
    }
    
    parameters {
        string(name: 'REPO_URL', defaultValue: 'https://github.com/Shivanshu332/terraformAWSdemo.git', description: 'GitHub repository URL')
        string(name: 'BRANCH', defaultValue: 'test', description: 'Branch name to checkout')
        string(name: 'AWS_Region', defaultValue: 'ap-south-1', description: 'AWS region')
        choice(name: 'environment', choices: ['dev', 'sit', 'uat', 'pre', 'prod'], description: 'Choose the environment to deploy')
        choice(name: 'Terraform_Destroy', choices: ['false', 'true'], description: 'Destroy Terraform resources?')
    }

    environment {
        AWS_DEFAULT_REGION = "${params.AWS_Region}"
        TF_VAR_file = "terraform.tfvars"  
    }

    stages {
        stage('Checkout Code') {
            steps {
                script {
                    echo "Cloning repository: ${params.REPO_URL} (Branch: ${params.BRANCH})"
                    checkout([$class: 'GitSCM', branches: [[name: "*/${params.BRANCH}"]], userRemoteConfigs: [[url: params.REPO_URL]]])
                }
            }
        }

        stage('Verify Terraform Files') {
            steps {
                dir("environment/${params.environment}") {  
                    sh '''
                        set -e
                        echo "Checking if Terraform files exist..."
                        ls -lah
                    '''
                }
            }
        }

        stage('Terraform Init') {
            steps {
                dir("environment/${params.environment}") {  
                    sh '''
                        set -e
                        echo 'Initializing Terraform...'
                        terraform init
                    '''
                }
            }
        }
        
        stage('Terraform Plan') {
            when { expression { params.Terraform_Destroy == 'false' } }
            steps {
                dir("environment/${params.environment}") {  
                    sh '''
                        set -e
                        echo 'Generating Terraform plan...'
                        terraform plan -var-file=${TF_VAR_file} -out=tfplan
                        echo 'Terraform Plan Output:'
                        terraform show -no-color tfplan
                    '''
                }
            }
        }

        stage('Approval Apply') {
            when { expression { params.Terraform_Destroy == 'false' } }
            steps {
                script {
                    timeout(time: 15, unit: 'MINUTES') {
                        input message: 'Approve Terraform Apply?', ok: 'Proceed'
                    }
                }
            }
        }

        stage('Terraform Apply') {
            when { expression { params.Terraform_Destroy == 'false' } }
            steps {
                dir("environment/${params.environment}") { 
                    sh '''
                        set -e
                        echo 'Applying Terraform changes...'
                        terraform apply -auto-approve tfplan
                    '''
                }
            }
        }
        
        stage('Terraform Destroy Plan') {
            when { expression { params.Terraform_Destroy == 'true' } }
            steps {
                dir("environment/${params.environment}") {  
                    sh '''
                        set -e
                        echo 'Generating Terraform Destroy Plan...'
                        terraform plan -destroy -out=tfplandestroy -var-file=${TF_VAR_file} | tee destroy_plan.log
                        echo 'Terraform Destroy Plan Output:'
                        terraform show -no-color tfplandestroy
                    '''
                }
            }
        }

        stage('Approval Destroy') {
            when { expression { params.Terraform_Destroy == 'true' } }
            steps {
                script {
                    timeout(time: 15, unit: 'MINUTES') {
                        input message: 'Approve Terraform Destroy?', ok: 'Proceed'
                    }
                }
            }
        }

        stage('Terraform Destroy Apply') {
            when { expression { params.Terraform_Destroy == 'true' } }
            steps {
                dir("environment/${params.environment}") { 
                    sh '''
                        set -e
                        echo 'Destroying Terraform infrastructure...'
                        terraform apply -auto-approve tfplandestroy
                    '''
                }
            }
        }
    }
}
