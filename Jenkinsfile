pipeline {
  agent {
    kubernetes {
      yamlFile 'jenkins-agent.yaml'
      defaultContainer 'bash'
      idleMinutes 1
    }
  }

  environment {
    TARGET = '34.116.148.25'
    CREDENTIAL_ID = 'jenkins-ssh-key-for-host1'
  }

  stages {
    stage('Scan Before Hardening') {
      environment {
        INSPEC_LINUX_BASE_PROFILE = 'https://github.com/dev-sec/linux-baseline'
      }
      steps {
        container('inspec') {
          withCredentials([sshUserPrivateKey(credentialsId: "${env.CREDENTIAL_ID}", \
                                             keyFileVariable: 'SSH_KEY_FOR_TARGET', \
                                             usernameVariable: 'SSH_USER_FOR_TARGET')]) {
            // catchError(buildResult: 'SUCCESS', stageResult: 'FAILURE') {
            //   sh '''
            //     inspec exec ${INSPEC_LINUX_BASE_PROFILE} \
            //       --chef-license=accept \
            //       --target=ssh://${SSH_USER_FOR_TARGET}@${TARGET} \
            //       -i ${SSH_KEY_FOR_TARGET} \
            //       --no-distinct-exit
            //   '''
            // }
            script {
              def scanResult = sh(script: '''
                inspec exec ${INSPEC_LINUX_BASE_PROFILE} \
                  --chef-license=accept \
                  --target=ssh://${SSH_USER_FOR_TARGET}@${TARGET} \
                  -i ${SSH_KEY_FOR_TARGET} \
                  --no-distinct-exit
              ''', returnStatus: true
              )
              if (scanResult =! 0) {
                currentBuild.result = 'FAILURE'
                error "Scan before hardening failed!"
              }
            }
          }
        }
      }
    }

    stage('Pause/Review') {
      when {
        expression { currentBuild.result == 'FAILURE' }
      }
      steps {
        input message: 'Inspec scan failed. Do you want to continue with Hardening?', ok: 'Proceed', parameters: [
          string(defaultValue: '', description: 'Enter any comments or remediation action taken', name: 'Remediation Comments')
        ]
      }
    }

    stage('Hardening') {
      when {
        expression { currentBuild.result == 'FAILURE'}
      }
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

    stage('Scan After Hardening') {
      when {
        expression { currentBuild.result == 'FAILURE'}
      }
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
