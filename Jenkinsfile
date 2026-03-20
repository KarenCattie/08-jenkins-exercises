@Library('NodeJS-jenkins-shared-library')_
pipeline {
    agent any // run on any available Jenkins agent

    tools { // what tools Jenkins should make available
        nodejs 'node-24'
    }

    environment {   // global variables available to all stages
        IMAGE_NAME = 'catdomeow/08-jenkins-exercises'
    }

    stages { // the actual pipeline steps
        stage('Increment Version') {
            steps {
                script {
                    incrementVersion()
                }
            }
        }
        stage('Test') {
            steps {
                script {
                    runTests()
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                script {
                    buildImage(IMAGE_NAME)
                }
            }
        }
        stage('Push to Docker Hub') {
            steps {
                script {
                    dockerLogin()
                    dockerPush(IMAGE_NAME)
                }
            }
        }
        stage('Commit to Git') {
            steps {
                script {
                    commitToGit()
                }
            }
        }
    }
}
