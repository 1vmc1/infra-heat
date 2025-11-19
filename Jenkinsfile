pipeline {
  agent { label 'vmc2' }

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
          // Включаем вывод команд для отладки
          sh 'set -x'

          // Проверяем, существует ли стек
          def stackExists = sh(
            script: "openstack stack list -f value -c 'Stack Name' | grep -w ${env.STACK_NAME} || true",
            returnStdout: true
          ).trim()
          echo "Stack exists: ${stackExists}"

          if (stackExists) {
            sh """
              openstack stack update -t server.yaml \\
                --parameter key_name=${params.KEY_NAME} \\
                --parameter image=${params.IMAGE} \\
                --parameter flavor=${params.FLAVOR} \\
                ${env.STACK_NAME}
            """
          } else {
            sh """
              openstack stack create -t server.yaml \\
                --parameter key_name=${params.KEY_NAME} \\
                --parameter image=${params.IMAGE} \\
                --parameter flavor=${params.FLAVOR} \\
                ${env.STACK_NAME}
            """
          }

          // Ждём завершения операции
          sh "openstack stack wait ${env.STACK_NAME}"

          // Получаем IP сервера из вывода стека
          def serverIp = sh(
            script: "openstack stack output show ${env.STACK_NAME} server_ip -f value",
            returnStdout: true
          ).trim()
          echo "Server IP: ${serverIp}"
        }
      }
    }
  }
}
