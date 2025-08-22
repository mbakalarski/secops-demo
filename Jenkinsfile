pipeline {
  agent {
    kubernetes {
      yamlFile 'jenkins-agent.yaml'
      defaultContainer 'bash'
      idleMinutes 1
    }
  }

  environment {
    TARGET = '34.118.72.127'
    CREDENTIAL_ID = 'jenkins-ssh-key-for-host1'
  }

  stages {
    stage('Hardening') {
      steps {
        container('ansible') {
          withCredentials([sshUserPrivateKey(credentialsId: "${env.CREDENTIAL_ID}", \
                                             keyFileVariable: 'SSH_KEY_FOR_TARGET', \
                                             usernameVariable: 'SSH_USER_FOR_TARGET')]) {
            sh '''
              ansible-galaxy collection install devsec.hardening && \
              ansible-playbook -i "${TARGET}," \
                               --ssh-extra-args '-o StrictHostKeyChecking=no' \
                               -u ${SSH_USER_FOR_TARGET} \
                               --private-key ${SSH_KEY_FOR_TARGET} \
                               ./ansible/hardening.yaml
            '''
          }
        }
      }
    }

    stage('Compliance Run') {
      environment {
        INSPEC_LINUX_BASE_PROFILE = 'https://github.com/dev-sec/linux-baseline'
      }
      steps {
        container('inspec') {
          withCredentials([sshUserPrivateKey(credentialsId: "${env.CREDENTIAL_ID}", \
                                             keyFileVariable: 'SSH_KEY_FOR_TARGET', \
                                             usernameVariable: 'SSH_USER_FOR_TARGET')]) {
            sh '''
              inspec exec ${INSPEC_LINUX_BASE_PROFILE} \
                --chef-license=accept \
                --target=ssh://${SSH_USER_FOR_TARGET}@${TARGET} \
                -i ${SSH_KEY_FOR_TARGET} \
                --waiver-file=./inspec/waiver.yaml \
                --no-distinct-exit
            '''
          }
        }
      }
    }
  }
}
