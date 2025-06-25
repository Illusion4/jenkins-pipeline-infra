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
    booleanParam(name: 'RUN_PLAN', defaultValue: true, description: 'Run terraform plan before apply?')
    booleanParam(name: 'AUTO_APPROVE', defaultValue: true, description: 'Auto-approve Terraform apply?')
  }

  environment {
    TF_VAR_cloud_bucket       = credentials('tf-cloud-bucket')
    TF_VAR_cloudflare_api_token   = credentials('cloudflare-token')
    TF_VAR_gcp_credentials_file = credentials('gcp-sa-key')
    TF_VAR_jenkins_github_ssh_private_key = "."
    GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp-sa-key')
  }

  stages {
    stage('Clone Repositories') {
            steps {
                dir('config') {
                    git branch: params.config_branch, url: params.config_repo
                    sh 'ls -la'
                    sh 'pwd'
                }
                dir('infra') {
                    git branch: 'jenkins-infra-pipeline', url: 'https://github.com/iviul/Milestone-2.git'
                    sh 'ls -la'
                    sh 'pwd'
                }
                dir('infra/terraform') {
                    sh 'cp "../../config/${params.config_file}" .'
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
      when {
        expression { return params.RUN_PLAN }
      }
      steps {
        dir('infra/terraform/gcp') {
          sh "terraform plan -out=tfplan"
        }
      }
    }

    stage('Terraform Apply') {
      steps {
        dir('infra/terraform/gcp') {
          sh """
            terraform apply ${params.AUTO_APPROVE ? '-auto-approve' : 'tfplan'}
          """
        }
      }
    }
  }
}
