pipeline {
  agent any

  environment {
    OS_CLOUD = 'mycloud'
    STACK_NAME = 'project-stack'
  }

  parameters {
    string(name: 'KEY_NAME', defaultValue: 'vmc', description: 'SSH Key Name')
    string(name: 'IMAGE', defaultValue: 'ubuntu-22.04', description: 'Image Name')
    string(name: 'FLAVOR', defaultValue: 'm1.small', description: 'Flavor')
  }

  stages {
    stage('Checkout') {
      steps {
        git 'git@github.com:1vmc1/infra-heat.git'
      }
    }

    stage('Deploy Stack') {
      steps {
        script {
          def stackExists = sh(script: "openstack stack list -f value -c 'Stack Name' | grep -w ${STACK_NAME} || true", returnStdout: true).trim()
          if (stackExists) {
            sh """
              openstack stack update -t server.yaml \\
                --parameter key_name=${params.KEY_NAME} \\
                --parameter image=${params.IMAGE} \\
                --parameter flavor=${params.FLAVOR} \\
                ${STACK_NAME}
            """
          } else {
            sh """
              openstack stack create -t server.yaml \\
                --parameter key_name=${params.KEY_NAME} \\
                --parameter image=${params.IMAGE} \\
                --parameter flavor=${params.FLAVOR} \\
                ${STACK_NAME}
            """
          }
          sh "openstack stack wait ${STACK_NAME}"
          def serverIp = sh(script: "openstack stack output show ${STACK_NAME} server_ip -f value", returnStdout: true).trim()
          echo "Server IP: ${serverIp}"
        }
      }
    }
  }
}
