pipeline {
  agent {
    kubernetes {
      yamlFile 'jenkins-agent.yaml'
      defaultContainer 'bash'
      idleMinutes 1
    }
  }

  stages {
    stage('Compliance Run') {
      environment {
        HOST = '34.118.72.127'
        INSPEC_LINUX_BASE_PROFILE = 'https://github.com/dev-sec/linux-baseline'
        CREDENTIAL_ID = 'jenkins-ssh-key-for-host1'
      }
      steps {
        container('inspec') {
          withCredentials([sshUserPrivateKey(credentialsId: $CREDENTIAL_ID, \
                                             keyFileVariable: 'SSH_KEY_FOR_HOST', \
                                             usernameVariable: 'SSH_USER_FOR_HOST')]) {
            sh '''
              inspec exec ${INSPEC_LINUX_BASE_PROFILE} \
                --target=ssh://${SSH_USER_FOR_HOST}@${HOST} \
                -i $SSH_KEY_FOR_HOST --chef-license=accept
            '''
          }
        }
      }
    }
  }
}
