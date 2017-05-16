#!/usr/bin/env groovy

node {
    stage('checkout') {
        checkout scm
    }

    stage('check java') {
        sh "java -version"
    }

    stage('clean') {
        sh "chmod +x gradlew"
        sh "./gradlew clean --no-daemon"
    }


    stage('backend tests') {
        try {
            sh "./gradlew test -PnodeInstall --no-daemon"
        } catch(err) {
            throw err
        } finally {
            junit '**/build/**/TEST-*.xml'
        }
    }

    stage('packaging') {
        sh "./gradlew bootRepackage -x test -Pprod -PnodeInstall --no-daemon"
        archiveArtifacts artifacts: '**/build/libs/*.war', fingerprint: true
    }

    stage('deployment') {
        sh "./gradlew deployHeroku --no-daemon"
    }


    def dockerImage
    stage('build docker') {
        sh "cp -R src/main/docker build/"
        sh "cp build/libs/*.war build/docker/"
        dockerImage = docker.build('microapp', 'build/docker')
    }

    stage('publish docker') {
        docker.withRegistry('https://hub.docker.com/vnayakdocker', 'vnayakdocker') {
            dockerImage.push 'latest'
        }
    }
}
