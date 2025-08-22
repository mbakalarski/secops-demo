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
          withCredentials([sshUserPrivateKey(credentialsId: "${env.CREDENTIAL_ID}", \
                                             keyFileVariable: 'SSH_KEY_FOR_HOST', \
                                             usernameVariable: 'SSH_USER_FOR_HOST')]) {
            sh '''
              cat <<EOT > /share/waiver.yaml
              os-12:
                justification: "Not applicable in this environment"
                run: false
              EOT
              
              inspec exec ${INSPEC_LINUX_BASE_PROFILE} \
                --target=ssh://${SSH_USER_FOR_HOST}@${HOST} \
                -i $SSH_KEY_FOR_HOST --chef-license=accept \
                --waiver-file=/share/waiver.yaml
            '''
          }
        }
      }
    }
  }
}
