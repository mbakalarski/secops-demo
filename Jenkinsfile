pipeline {
  agent {
    kubernetes {
      yamlFile 'jenkins-agent.yaml'
      defaultContainer 'bash'
      idleMinutes 1
    }
  }

  environment {
    HOST = '34.118.72.127'
    CREDENTIAL_ID = 'jenkins-ssh-key-for-host1'
  }

  stages {
    stage('Hardening') {
      steps {
        container('ansible') {
          withCredentials([sshUserPrivateKey(credentialsId: "${env.CREDENTIAL_ID}", \
                                             keyFileVariable: 'SSH_KEY_FOR_HOST', \
                                             usernameVariable: 'SSH_USER_FOR_HOST')]) {
            sh '''
              ansible-galaxy collection install devsec.hardening && \
              ansible-playbook -i ./ansible/hosts.ini \
                               --ssh-extra-args StrictHostKeyChecking=no \
                               -u ${SSH_USER_FOR_HOST} \
                               --private-key ${SSH_KEY_FOR_HOST} \
                               ./ansible/hardening.yaml
            '''
          }
        }
      }
    }

    stage('Compliance Run') {
      environment {
        // HOST = '34.118.72.127'
        INSPEC_LINUX_BASE_PROFILE = 'https://github.com/dev-sec/linux-baseline'
        // CREDENTIAL_ID = 'jenkins-ssh-key-for-host1'
      }
      steps {
        container('inspec') {
          withCredentials([sshUserPrivateKey(credentialsId: "${env.CREDENTIAL_ID}", \
                                             keyFileVariable: 'SSH_KEY_FOR_HOST', \
                                             usernameVariable: 'SSH_USER_FOR_HOST')]) {
            sh '''
              inspec exec ${INSPEC_LINUX_BASE_PROFILE} \
                --chef-license=accept \
                --target=ssh://${SSH_USER_FOR_HOST}@${HOST} \
                -i $SSH_KEY_FOR_HOST \
                --waiver-file=./inspec/waiver.yaml \
                --no-distinct-exit
            '''
          }
        }
      }
    }
  }
}
