pipeline {
  agent {
        kubernetes {
            label 'terraform-agent'
            defaultContainer 'terraform'
            containerTemplate {
                name 'terraform'
                image 'vladyslavlintur/terraform-gcloud:latest'
                ttyEnabled true
                command 'cat'
            }
            namespace 'jenkins'
        }
    }

  parameters {
    string(name: 'config_repo', defaultValue: 'https://github.com/Illusion4/jenkins-pipeline-infra.git', description: 'Config repository URL')
    string(name: 'config_branch', defaultValue: 'main', description: 'Config branch')
    string(name: 'config_file', defaultValue: 'config-kuber.json', description: 'Path to JSON config file to use')
  }

  environment {
    TF_VAR_cloud_bucket       = credentials('cloud_bucket')
    TF_VAR_cloudflare_api_token   = credentials('cloudflare-token')
    TF_VAR_gcp_credentials_file = credentials('gcp-sa-key')
    TF_VAR_JENKINS_GITHUB_SSH_PRIVATE_KEY = credentials('ssh_privatekey_github')
    GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-sa-key')
  }

  stages {
    stage('Clone Repositories') {
            steps {
                dir('config') {
                    git branch: params.config_branch, credentialsId: 'ssh_privatekey_github', url: params.config_repo
                    sh 'ls -la'
                    sh 'pwd'
                }
                dir('infra') {
                    git branch: 'jenkins-infra-pipeline', url: 'https://github.com/iviul/Milestone-2.git'
                    sh 'ls -la'
                    sh 'pwd'
                }
                dir('infra/terraform') {
                    sh "cp \"../../config/${params.config_file}\" ."
                }
            }
        }
    stage('Terraform Init') {
      steps {
        dir('infra/terraform/gcp') {
          sh '''
            terraform init -backend-config="bucket=$TF_VAR_cloud_bucket" -backend-config="credentials=$TF_VAR_gcp_credentials_file" -reconfigure
          '''
        }
      }
    }

     stage('Terraform Plan') {
      steps {
        dir('infra/terraform/gcp') {
          sh 'terraform plan -out=tfplan -no-color'
        }
      }
    }

    stage('Terraform Apply') {
      steps {
        input(
            message: "Approve Terraform Apply?", 
            ok: "Apply Now",
            id: "terraform-approval"
        )
        
        dir('infra/terraform/gcp') {          
          sh 'terraform apply -auto-approve -no-color tfplan'
        }
      }
    }
    
  }
}
