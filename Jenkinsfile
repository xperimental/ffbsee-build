pipeline {
  agent any
  stages {
    stage('checkout tools') {
      steps {
        checkout scm
      }
    }

    stage('checkout') {
      steps {
        checkout([
          $class: 'GitSCM',
          branches: [[name: 'v17.01.2']],
          doGenerateSubmoduleConfigurations: false,
          extensions: [
            [$class: 'RelativeTargetDirectory', relativeTargetDir: 'source']
          ],
          submoduleCfg: [],
          userRemoteConfigs: [
            [url: 'git://git.lede-project.org/source.git']
          ]
        ])
        checkout([
          $class: 'GitSCM',
          branches: [[name: 'dev']],
          doGenerateSubmoduleConfigurations: false,
          extensions: [
            [$class: 'RelativeTargetDirectory', relativeTargetDir: 'source/firmware']
          ],
          submoduleCfg: [],
          userRemoteConfigs: [
            [url: 'git://github.com/ffbsee/firmware.git']
          ]
        ])
      }
    }

    stage('feeds') {
      steps {
        dir("source") {
          sh "./scripts/feeds update -a"
          sh "./scripts/feeds install -a"
        }
      }
    }


    stage('patches') {
      steps {
        dir("source") {
          sh "cp -rf firmware/files firmware/package ."
          sh 'for file in firmware/patches/lede/*.patch; do patch -p1 < $file; done'
          dir('feeds/routing') {
            sh 'for file in ../../firmware/patches/routing/*.patch; do patch -p1 < $file; done'
          }
        }
      }
    }

    stage('build') {
      steps {
        dir("source") {
          sh "../build.sh"
        }
      }
    }
  }
}
