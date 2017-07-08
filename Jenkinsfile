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
        sh 'for file in firmware/patches/lede/*.patch; do patch -p1 < $file; done'
        dir('feeds/routing') {
            sh 'for file in ../../firmware/patches/routing/*.patch; do patch -p1 < $file; done'
        }
    }

    stage('build') {
        sh '''#!/bin/sh

platforms='
	CONFIG_TARGET_arm64=y
	CONFIG_TARGET_ath25=y
	CONFIG_TARGET_ar71xx=y
	CONFIG_TARGET_brcm2708=y\nCONFIG_TARGET_brcm2708_bcm2708=y
	CONFIG_TARGET_brcm2708=y\nCONFIG_TARGET_brcm2708_bcm2709=y
	CONFIG_TARGET_bcm53xx=y
	CONFIG_TARGET_brcm47xx=y
	CONFIG_TARGET_ramips=y\nCONFIG_TARGET_ramips_rt305x=y
	CONFIG_TARGET_ramips=y\nCONFIG_TARGET_ramips_mt7620=y
	CONFIG_TARGET_ramips=y\nCONFIG_TARGET_ramips_mt7621=y
	CONFIG_TARGET_ramips=y\nCONFIG_TARGET_ramips_mt7628=y
	CONFIG_TARGET_ramips=y\nCONFIG_TARGET_ramips_rt3883=y
	CONFIG_TARGET_ramips=y\nCONFIG_TARGET_ramips_rt288x=y
'

for platform in $platforms; do
	echo "$platform" > .config

	echo "CONFIG_TARGET_MULTI_PROFILE=y" >> .config
	echo "CONFIG_TARGET_ALL_PROFILES=y" >> .config
	echo "CONFIG_TARGET_PER_DEVICE_ROOTFS=y" >> .config
	echo "CONFIG_PACKAGE_freifunk-basic=y" >> .config

	# Debug output
	echo "Build: $platform"

	# Build image
	make defconfig
	make -j4
done
'''
    }
}
