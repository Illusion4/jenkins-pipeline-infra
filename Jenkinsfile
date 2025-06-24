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
    booleanParam(name: 'AUTO_APPROVE', defaultValue: true, description: 'Auto-approve?')
  }

  environment {
    TF_VAR_cloud_bucket       = credentials('tf-cloud-bucket')
    TF_VAR_cloudflare_api_token   = credentials('cloudflare-token')
    TF_VAR_gcp_credentials_file = credentials('gcp-sa-key')
    TF_VAR_jenkins_github_ssh_private_key = "."
  }

  stages {
    stage('Clone Repositories') {
            steps {
                dir('config') {
                    git branch: 'main', url: 'https://github.com/Illusion4/jenkins-pipeline-infra.git'
                    sh 'ls -la'
                    sh 'pwd'
                }
                dir('infra') {
                    git branch: 'jenkins-infra-pipeline', url: 'https://github.com/iviul/Milestone-2.git'
                    sh 'ls -la'
                    sh 'pwd'
                }
                dir('infra/terraform') {
                    sh 'cp "../../config/config-kuber.json" .'
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

    stage('Terraform Apply') {
      steps {
          dir('infra/terraform/gcp') {
          sh "terraform apply ${params.AUTO_APPROVE ? '-auto-approve' : ''}"
        }
      }
    }
  }
}
