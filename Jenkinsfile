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
                dir('app'){ // runs the command inside the app/ folder
                    sh 'npm version minor --no-git-tag-version' // npm version minor - bumps the middle number in package.json e.g. 1.0.0 → 1.1.0
                                                                // --no-git-tag-version — prevents npm from trying to create a git tag (Jenkins handles git itself, so this would cause a conflict)
                }
                script {
                    def version = sh(
                        script: "cd app && node -p \"require('./package.json').version\"",
                        returnStdout:true // runs the command AND captures its output as a string
                    ).trim() // removes any trailing newline from the output
                    env.IMAGE_VERSION = "$version-$BUILD_NUMBER" // stores it as a pipeline environment variable so later stages can use ${IMAGE_VERSION}
                }
            }
        }
        stage('Test') {
            steps {
                dir('app'){
                    sh 'npm install' // Installs dependencies then runs Jest tests
                    sh 'npm test'    // If npm test fails, the entire pipeline stops here — nothing gets built or pushed. This is the safety gate that ensures only working code gets deployed
                }
            }
        }
        stage('Build Docker Image') {
            steps {
                sh "docker build -t ${IMAGE_NAME}:${IMAGE_VERSION} ." // Builds the Docker image using Dockerfile that is located in the project root
            }
        }
        stage('Push to Docker Hub') {
            steps {
                withCredentials([usernamePassword( // withCredentials — securely injects the credentials that stored in Jenkins. The password is masked in logs so it never appears as plain text
                    credentialsId: 'docker-hub-creds',
                    usernameVariable: 'DOCKER_USER',
                    passwordVariable: 'DOCKER_PASS'
                )]) {
                    sh "echo ${DOCKER_PASS} | docker login -u ${DOCKER_USER} --password-stdin"
                    sh "docker push ${IMAGE_NAME}:${IMAGE_VERSION}"
                    }
            }
        }
        stage('Commit to Git') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-creds',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh 'git config --global user.email "jenkins@ci.com"' // Sets a git identity label for Jenkins (required to make commits)
                    sh 'git config --global user.name "Jenkins"'
                    sh "git remote set-url origin https://${GIT_USER}:${GIT_PASS}@github.com/KarenCattie/08-jenkins-exercises.git"
                    sh 'git add app/package.json' // Only commits `package.json` — that's the only file that changed (the version bump)
                    sh "git commit -m \"ci: bump version to ${IMAGE_VERSION}\""
                    sh "git push origin HEAD:main" // HEAD = "my current local commit", main = "push it to the main branch on GitHub"
                }
            }
        }
    }
}
