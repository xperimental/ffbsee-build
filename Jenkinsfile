node ("make") {
    stage('checkout') {
        checkout([$class: 'GitSCM', branches: [[name: 'v17.01.2']], doGenerateSubmoduleConfigurations: false, extensions: [], submoduleCfg: [], userRemoteConfigs: [[url: 'git://git.lede-project.org/source.git']]])
        checkout([$class: 'GitSCM', branches: [[name: 'dev']], doGenerateSubmoduleConfigurations: false, extensions: [[$class: 'RelativeTargetDirectory', relativeTargetDir: 'firmware']], submoduleCfg: [], userRemoteConfigs: [[url: 'git://github.com/ffbsee/firmware.git']]])
    }

    stage('feeds') {
        sh "./scripts/feeds update -a"
        sh "./scripts/feeds install -a"
    }

    stage('patches') {
        sh "cp -rf firmware/files firmware/package ."
        sh "git am --whitespace=nowarn firmware/patches/lede/*.patch"
        dir('feeds/routing') {
            sh "git am --whitespace=nowarn ../../firmware/patches/routing/*.patch"
        }
    }

    stage('build') {
        sh "make defconfig"
    }
}
