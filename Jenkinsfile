#!/usr/bin/env groovy

def shell(command) {
    return bat(returnStdout: true, script: "sh -x -c \"${command}\"").trim()
}

node {
    stage('checkout') {
        checkout scm
    }

    stage('check java') {
        shell("java -version")
    }

    stage('clean') {
        shell("chmod +x mvnw")
        shell("mvnw clean")
    }

    stage('install tools') {
        bat "mvnw com.github.eirslett:frontend-maven-plugin:install-node-and-npm -DnodeVersion=v6.11.1 -DnpmVersion=5.3.0"
    }

    stage('npm install') {
        bat "mvnw com.github.eirslett:frontend-maven-plugin:npm"
    }

    stage('backend tests') {
        try {
            bat "mvnw test"
        } catch(err) {
            throw err
        } finally {
            junit '**/target/surefire-reports/TEST-*.xml'
        }
    }

    stage('frontend tests') {
        try {
            bat "mvnw com.github.eirslett:frontend-maven-plugin:npm -Dfrontend.yarn.arguments=test"
        } catch(err) {
            throw err
        } finally {
            junit '**/target/test-results/karma/TESTS-*.xml'
        }
    }

    stage('packaging') {
        bat "mvnw package -Pprod -DskipTests"
        archiveArtifacts artifacts: '**/target/*.war', fingerprint: true
    }

    stage('quality analysis') {
        withSonarQubeEnv('Sonar') {
            bat "mvnw sonar:sonar"
        }
    }
}
