pipeline {
  agent any

  parameters {
    string(name: 'RELEASE_TAG', defaultValue: '', description: 'Optional release tag (e.g. v1.0.0)')
    string(name: 'ROLLBACK_TO', defaultValue: '', description: 'If set, rollback to this version')
  }

  environment {
    ANSIBLE_CONTROLLER = 'ec2-user@16.16.201.56'
    ANSIBLE_KEY_PATH = '/var/lib/jenkins/.ssh/smkey.pem'
    WORKSPACE_ARTIFACTS = "${env.WORKSPACE}/artifacts"
  }

  stages {
    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Build') {
      when {
        expression { params.ROLLBACK_TO == '' }
      }
      steps {
        dir('frontend') {   // ← change this to the correct folder name
          sh 'npm install'
          sh 'npm run build -- --prod'
        }
      }
    }

    stage('Package') {
      when {
        expression { params.ROLLBACK_TO == '' }
      }
      steps {
        script {
          def release = params.RELEASE_TAG?.trim() ?: "v${new Date().format('yyyyMMddHHmmss')}"
          env.RELEASE_VERSION = release
          sh "mkdir -p ${WORKSPACE_ARTIFACTS}"
          dir('frontend/dist') {
            sh "tar -czf ${WORKSPACE_ARTIFACTS}/angular-${release}.tar.gz ."
          }
        }
      }
      post {
        success {
          archiveArtifacts artifacts: 'artifacts/*.tar.gz', fingerprint: true
        }
      }
    }

    stage('DeployOrRollback') {
      steps {
        script {
          if (params.ROLLBACK_TO?.trim()) {
            sh "ssh -o StrictHostKeyChecking=no -i ${ANSIBLE_KEY_PATH} ${ANSIBLE_CONTROLLER} 'cd ~/ansible-project && ansible-playbook playbooks/deploy_angular.yml -e \\\"artifact_name=angular-${params.ROLLBACK_TO}.tar.gz release_version=${params.ROLLBACK_TO} artifact_local_path=~/ansible-project/artifacts/angular-${params.ROLLBACK_TO}.tar.gz\\\"'"
          } else {
            sh "scp -o StrictHostKeyChecking=no -i ${ANSIBLE_KEY_PATH} ${WORKSPACE_ARTIFACTS}/angular-${env.RELEASE_VERSION}.tar.gz ec2-user@16.16.201.56:~/ansible-project/artifacts/"
            sh "ssh -o StrictHostKeyChecking=no -i ${ANSIBLE_KEY_PATH} ec2-user@16.16.201.56 'cd ~/ansible-project && ansible-playbook playbooks/deploy_angular.yml -e \\\"artifact_name=angular-${env.RELEASE_VERSION}.tar.gz release_version=${env.RELEASE_VERSION} artifact_local_path=~/ansible-project/artifacts/angular-${env.RELEASE_VERSION}.tar.gz\\\"'"
          }
        }
      }
    }
  }

  post {
    always {
      echo 'Pipeline finished.'
    }
  }
}
