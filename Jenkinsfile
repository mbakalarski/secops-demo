pipeline {
  agent {
    kubernetes {
      yamlFile 'jenkins-agent.yaml'
      defaultContainer 'bash'
      idleMinutes 1
    }
  }
  environment {
    // HOST1_INSPECT_TARGET = 'ssh://student@34.116.253.244'
    HOST1 = '34.116.253.244'
    INSPEC_LINUX_BASE_PROFILE = 'https://github.com/dev-sec/linux-baseline'
  }

  stages {
    stage('Compliance Run') {
      steps {
        container('inspec') {
            script {
              withCredentials([sshUserPrivateKey(credentialsId: 'sshUser', keyFileVariable: 'SSH_PRIVATE_KEY', usernameVariable: 'SSH_USERNAME')]) {
                writeFile file: '/tmp/ssh-key', text: SSH_PRIVATE_KEY
                sh '''
                  inspec exec ${INSPEC_LINUX_BASE_PROFILE} --target=ssh://${SSH_USERNAME}@${HOST1} -i /tmp/ssh-key --chef-license=accept
                '''
              }
            }
        }
      }
    }
  }

  /*
  stages {
    stage('Daily Compliance Run') {
      steps{
        echo 'Running a compliance scan with inspec....'
          script{
            def remote = [:]
            remote.name = "controlnode"
            remote.host = "34.116.253.244"
            remote.allowAnyHosts = true

            withCredentials([sshUserPrivateKey(credentialsId: 'sshUser', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')]) {
                remote.user = userName
                remote.identityFile = identity
                stage("Placeholder Stage...") {
                  sshCommand remote: remote, sudo: true, command: 'echo "add your stuff here....."'
                  sshCommand remote: remote, sudo: true, command: 'echo "some more stuff goes here....."'
              }
                stage("Scan with InSpec") {
                  sshCommand remote: remote, sudo: true, command: 'inspec exec /root/linux-baseline/'
              }
            }
          }
       }
    }
  }
  */
}
