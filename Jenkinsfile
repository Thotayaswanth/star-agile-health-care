pipeline {
  agent any

  stages {
    stage('Git Checkout') {
      steps {
        echo 'This stage is to clone the repo from github'
        git branch: 'master', url: 'https://github.com/Thotayaswanth/star-agile-health-care.git'
      }
    }

    stage('Create Package') {
      steps {
        echo 'This stage will compile, test, package my application'
        sh 'mvn package'
      }
    }

    stage('AWS-Login') {
      steps {
        withCredentials([aws(
          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
          credentialsId: 'awslogin',
          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        )]) {
          echo 'AWS credentials set'
        }
      }
    }

    stage('Setting the Kubernetes Cluster') {
      steps {
        withCredentials([aws(
          credentialsId: 'awslogin',
          accessKeyVariable: 'AWS_ACCESS_KEY_ID',
          secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
        )]) {
          dir('terraform_files') {
            sh 'terraform init'
            sh 'terraform validate'
            sh 'terraform apply --auto-approve'
            sh 'sleep 20'
          }
        }
      }
    }

    stage('Deploy to Kubernetes') {
      steps {
        sh 'chmod 600 ./terraform_files/kav.pem'   
       
        sh 'scp -o StrictHostKeyChecking=no -i ./terraform_files/kav.pem deployment.yml ubuntu@172.31.22.130:/home/ubuntu/'
        sh 'scp -o StrictHostKeyChecking=no -i ./terraform_files/kav.pem service.yml ubuntu@172.31.22.130:/home/ubuntu/'

        script {
          try {
            sh 'ssh -o StrictHostKeyChecking=no -i ./terraform_files/kav.pem ubuntu@172.31.22.130 kubectl apply -f .'
          } catch (error) {
            sh 'ssh -o StrictHostKeyChecking=no -i ./terraform_files/kav.pem ubuntu@172.31.22.130 kubectl apply -f .'
          }
        }
      }
    }
  }
}

