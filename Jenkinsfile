pipeline {
  agent { label 'terraform' }

  stages {
    stage('Clone Repositories') {
            steps {
                dir('config') {
                    git branch: 'DI-34-Develop', credentialsId: 'ssh_privatekey_github', url: 'git@github.com:iviul/Config.git'
                    sh 'ls -la'
                    sh 'pwd'
                }
                dir('milestone') {
                    git branch: 'test-jenkins-application', url: 'https://github.com/iviul/Milestone-2.git'
                    sh 'ls -la'
                    sh 'pwd'
                }
                dir('milestone/terraform') {
                    sh 'cp "../../config/config-kuber.json" .'
                }
            }
        }
    stage('Init') {
      steps {
        container('terraform') {
          sh 'terraform init'
        }
      }
    }
    stage('Apply') {
      steps {
        container('terraform') {
          sh 'terraform apply -auto-approve'
        }
      }
    }
  }
}
