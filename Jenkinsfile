pipeline {
  agent any
  triggers { pollSCM('* * * * *') }
  options { timestamps(); disableConcurrentBuilds() }

  environment {
    DEPLOY_HOST = '141.147.150.180'
    DEPLOY_DIR  = '/opt/spring-demo'
  }

  stages {
    stage('Checkout') {
      steps { checkout scm }
    }

    stage('Test & Build') {
      steps {
        sh 'chmod +x gradlew && ./gradlew clean test bootJar'
      }
    }

    stage('Deploy') {
    steps {
        withCredentials([sshUserPrivateKey(
            credentialsId: 'oci-vm-ssh',
            keyFileVariable: 'SSH_KEY',
            usernameVariable: 'SSH_USER'
        )]) {
            sh '''
                set -eu

                scp -i "$SSH_KEY" \
                  -o StrictHostKeyChecking=yes \
                  build/libs/app.jar \
                  "${SSH_USER}@${DEPLOY_HOST}:${DEPLOY_DIR}/app.jar.new"

                ssh -i "$SSH_KEY" \
                  -o StrictHostKeyChecking=yes \
                  "${SSH_USER}@${DEPLOY_HOST}" "
                    set -eu

                    if [ -f '${DEPLOY_DIR}/app.pid' ]; then
                      OLD_PID=\\$(cat '${DEPLOY_DIR}/app.pid')

                      if kill -0 \\"\\$OLD_PID\\" 2>/dev/null; then
                        kill \\"\\$OLD_PID\\"

                        for i in 1 2 3 4 5 6 7 8 9 10; do
                          if ! kill -0 \\"\\$OLD_PID\\" 2>/dev/null; then
                            break
                          fi
                          sleep 1
                        done
                      fi

                      rm -f '${DEPLOY_DIR}/app.pid'
                    fi

                    mv '${DEPLOY_DIR}/app.jar.new' \
                       '${DEPLOY_DIR}/app.jar'

                    nohup java -jar '${DEPLOY_DIR}/app.jar' \
                      --server.address=0.0.0.0 \
                      --server.port=8080 \
                      > '${DEPLOY_DIR}/app.log' 2>&1 < /dev/null &

                    echo \\$! > '${DEPLOY_DIR}/app.pid'
                  "
            '''
            }
        }
    }

    stage('Verify') {
      steps {
        retry(6) {
          sleep time: 5, unit: 'SECONDS'
          sh 'curl --fail --silent "http://${DEPLOY_HOST}:8080/health"'
        }
      }
    }
  }

  post {
    success { archiveArtifacts artifacts: 'build/libs/app.jar', fingerprint: true }
    failure { echo 'Jenkins 콘솔 출력과 VM journalctl 로그를 확인하세요.' }
  }
}