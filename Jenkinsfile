#!/usr/bin/env groovy

node {

  stage('Checkout') {
    checkout scm
  }

  stage('Build') {
    docker.image('zenoss/build-tools:0.0.10').inside('-u root') { 
      sh '''
        go get github.com/tools/godep
        $GOPATH/bin/godep go build -o redis-mon *.go
        make tgz
        
      '''
    }
  }

  stage('Publish app') {
    def remote = [:]
    withFolderProperties {
      withCredentials( [sshUserPrivateKey(credentialsId: 'PUBLISH_SSH_KEY', keyFileVariable: 'identity', passphraseVariable: '', usernameVariable: 'userName')] ) {
        remote.name = env.PUBLISH_SSH_HOST
        remote.host = env.PUBLISH_SSH_HOST
        remote.user = userName
        remote.identityFile = identity
        remote.allowAnyHosts = true

        def tar_ver = sh( returnStdout: true, script: "cat VERSION" ).trim()
        sshPut remote: remote, from: 'redis-mon-' + tar_ver + '.tgz', into: env.PUBLISH_SSH_DIR
      }
    }
  }
}
