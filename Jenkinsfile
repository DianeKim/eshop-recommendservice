pipeline {
  agent {
    kubernetes {
      yaml """
      kind: Pod
      spec:
        containers:
        - name: kaniko
          image: gcr.io/kaniko-project/executor:v1.6.0-debug
          command: 
          - cat
          tty: true
          """
    }
  }

  environment {
    SCP_CREDS = credentials('scpCredentials')
    DOCKER_REGISTRY = "<< SCR Private Endpoint URI >>"
  }
  stages {
    stage('Approval') {
      when {
        branch 'main'
      }
      steps {
        script {
          def plan = "${env.JOB_NAME} CI!!"
          input message: "Do you want to build and push?",
              parameters: [text(name: 'Plan', description: 'Please review the work', defaultValue: plan)]
        }
      }
    }
    stage('build and push docker image') {
      when {
        branch 'main'
      }
      steps {
        container('kaniko') {
          script {
            def dockerAuth = sh(returnStdout: true, script: "echo -n \"${SCP_CREDS_USR}:${SCP_CREDS_PSW}\" | base64").trim().replaceAll(/\n/, '')            
            sh """
              rm -rf /kaniko/.docker
              mkdir /kaniko/.docker
              echo '{\"auths\":{\"${DOCKER_REGISTRY}\":{\"auth\":\"$dockerAuth\"}}}' > /kaniko/.docker/config.json
              cat /kaniko/.docker/config.json
              /kaniko/executor \
                --git branch=main \
                --context=. \
                --destination=${DOCKER_REGISTRY}/eshop-recommendservice:latest
            """
          }
        }
      }
      post {
        success { 
          slackSend(channel: '<< slack채널 ID >>', color: 'good', message: "(Job:${env.JOB_NAME}- Build Number : ${env.BUILD_NUMBER}) CI success from <@<< 멤버ID >>>")
        }
        failure {
          slackSend(channel: '<< slack채널 ID >>', color: 'danger', message: "(Job:${env.JOB_NAME}- Build Number : ${env.BUILD_NUMBER}) CI fail from <@<< 멤버ID >>>")
        }
      }
    }
  }  
}
