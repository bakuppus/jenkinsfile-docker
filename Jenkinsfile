#!/usr/bin/env groovy

node {

    // Cleanup workspace
    deleteDir()

    //Clone example project from GitHub repository
    git url: 'https://github.com/bakuppus/jenkinsfile-docker.git', branch: 'master'
    def rtServer = Artifactory.server 'Jfrog'
    def rtDocker = Artifactory.docker server: rtServer
    def buildInfo = Artifactory.newBuildInfo()
    def ARTDOCKER_REGISTRY = 18.219.249.212:8081
    def tagDockerApp

    buildInfo.env.capture = true

    //Fetch all depedencies from Artifactory
    stage ('Dependencies') {
        dir ('.') {
            try {
                println "Gather Released Docker Framework and Gradle War file"
                def gradleWarDownload = """{
                    "files": [
                      {
                        "pattern": "maven-snapshot/*.war",
                        "target": "webservice.war",
                        "props": "unit-test=pass",
                        "flat": "true"
                      }
                    ]
                 }"""
                rtServer.download(gradleWarDownload, buildInfo )
            } catch (Exception e) {
                println "Caught Exception during resolution. Message ${e.message}"
                throw e
            }
        }
    }
    //Build docker image named docker-app
    stage ('Build & Deploy') {
        dir ('.') {
            tagDockerApp = "18.219.249.212:8081/docker-app:${env.BUILD_NUMBER}"
            println "Docker App Build"
            docker.build(tagDockerApp)
            println "Docker push" + tagDockerApp + " : " + REPO
            buildInfo = rtDocker.push(tagDockerApp, REPO, buildInfo)
            println "Docker Buildinfo"
            rtServer.publishBuildInfo buildInfo
        }
     }
 }
