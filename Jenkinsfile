pipeline {
    agent { label 'agent-1' }

    options {
        buildDiscarder logRotator(
            numToKeepStr: '10'
        )
    }
    environment {
        REPO_NAME = "plane"
        REPO_FULLNAME = "Plane"
        GITLAB_REP = "registry.gitlab.com/insightblue/${REPO_NAME}"
        PROJECT_PATH="/home/ubuntu/jenkins/workspace/abdul"
        SCRIPTS_K8S_PATH="${PROJECT_PATH}/deployment/k8s"
        TEST_BY = 'jenkins'
        GIT_BRANCH = "${env.GIT_BRANCH.replace('origin/', '')}"
        IMAGE_TAG = "${currentBuild.number}.${env.GIT_BRANCH.replace('origin/', '').replace('/', '-')}"
        DISCORD_WEBHOOK="https://discord.com/api/webhooks/1222242944384368782/VtgsxpyBMk-nQrlZhmyU0K8N4Xt6b3T1Cs5oYHYamJZrD2tYtBDmw75_0hpA6Ft3jJ0m"
        REPORT_LINK="https://jenkins.insightblue.co/job/abdul/${env.BUILD_NUMBER}/Test_20Reports/"
        TESTS_PATH="${PROJECT_PATH}/tests"
    }
    stages {
        stage('Cloning Git') {
            steps {
                script {
                    sh "git checkout ${env.GIT_BRANCH}"
                    sh "git reset --hard origin/${env.GIT_BRANCH}"

                    // Capture git commit message
                    env.GIT_COMMIT_MESSAGE = sh(
                        script: "git log -1 --pretty=format:'%s'",
                        returnStdout: true
                    ).trim()

                    // Capture git commit author
                    env.GIT_COMMIT_AUTHOR = sh(
                        script: "git log -1 --pretty=format:'%an'",
                        returnStdout: true
                    ).trim()

                    echo "Commit message: ${env.GIT_COMMIT_MESSAGE}"
                    echo "Commit author: ${env.GIT_COMMIT_AUTHOR}"
                }
            }
        }

        stage('Cleanup') {
            steps {
                script {
                    // Clean up docker system to free up space
                    // -a: Remove all unused images not just dangling ones
                    // -f: Force the prune without confirmation
                    sh "docker system prune -af || true"
                    sh "docker volume prune -f || true"
                }
            }
        }

        stage('Building Web') {
            steps{
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'khum38-gitlab',
                            usernameVariable: 'GITLAB_USERNAME',
                            passwordVariable: 'GITLAB_TOKEN'
                        ),
                        usernamePassword(
                            credentialsId: 'github-token',
                            usernameVariable: 'GITHUB_USERNAME',
                            passwordVariable: 'GITHUB_TOKEN'
                        )
                    ]) {
                        sh "docker login -u ${GITLAB_USERNAME} -p ${GITLAB_TOKEN} registry.gitlab.com"
                        sh """
                            docker build \
                                --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} \
                                --no-cache \
                                -f apps/web/Dockerfile.web \
                                -t ${GITLAB_REP}:${IMAGE_TAG}-web \
                                .
                        """
                        sh "docker push ${GITLAB_REP}:${IMAGE_TAG}-web"
                        sh "docker image rm -f ${GITLAB_REP}:${IMAGE_TAG}-web || true"
                    }
                }
            }
        }

        stage('Building Admin') {
            steps{
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'khum38-gitlab',
                            usernameVariable: 'GITLAB_USERNAME',
                            passwordVariable: 'GITLAB_TOKEN'
                        ),
                        usernamePassword(
                            credentialsId: 'github-token',
                            usernameVariable: 'GITHUB_USERNAME',
                            passwordVariable: 'GITHUB_TOKEN'
                        )
                    ]) {
                        sh "docker login -u ${GITLAB_USERNAME} -p ${GITLAB_TOKEN} registry.gitlab.com"
                        sh """
                            docker build \
                                --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} \
                                --no-cache \
                                -f apps/admin/Dockerfile.admin \
                                -t ${GITLAB_REP}:${IMAGE_TAG}-admin \
                                .
                        """
                        sh "docker push ${GITLAB_REP}:${IMAGE_TAG}-admin"
                        sh "docker image rm -f ${GITLAB_REP}:${IMAGE_TAG}-admin || true"
                    }
                }
            }
        }

        stage('Building Space') {
            steps{
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'khum38-gitlab',
                            usernameVariable: 'GITLAB_USERNAME',
                            passwordVariable: 'GITLAB_TOKEN'
                        ),
                        usernamePassword(
                            credentialsId: 'github-token',
                            usernameVariable: 'GITHUB_USERNAME',
                            passwordVariable: 'GITHUB_TOKEN'
                        )
                    ]) {
                        sh "docker login -u ${GITLAB_USERNAME} -p ${GITLAB_TOKEN} registry.gitlab.com"
                        sh """
                            docker build \
                                --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} \
                                --no-cache \
                                -f apps/space/Dockerfile.space \
                                -t ${GITLAB_REP}:${IMAGE_TAG}-space \
                                .
                        """
                        sh "docker push ${GITLAB_REP}:${IMAGE_TAG}-space"
                        sh "docker image rm -f ${GITLAB_REP}:${IMAGE_TAG}-space || true"
                    }
                }
            }
        }

        stage('Building API') {
            steps{
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'khum38-gitlab',
                            usernameVariable: 'GITLAB_USERNAME',
                            passwordVariable: 'GITLAB_TOKEN'
                        ),
                        usernamePassword(
                            credentialsId: 'github-token',
                            usernameVariable: 'GITHUB_USERNAME',
                            passwordVariable: 'GITHUB_TOKEN'
                        )
                    ]) {
                        sh "docker login -u ${GITLAB_USERNAME} -p ${GITLAB_TOKEN} registry.gitlab.com"
                        sh """
                            docker build \
                                --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} \
                                --no-cache \
                                -f apps/api/Dockerfile.api \
                                -t ${GITLAB_REP}:${IMAGE_TAG}-api \
                                ./apps/api
                        """
                        sh "docker push ${GITLAB_REP}:${IMAGE_TAG}-api"
                        sh "docker image rm -f ${GITLAB_REP}:${IMAGE_TAG}-api || true"
                    }
                }
            }
        }

        stage('Building Live') {
            steps{
                script {
                    withCredentials([
                        usernamePassword(
                            credentialsId: 'khum38-gitlab',
                            usernameVariable: 'GITLAB_USERNAME',
                            passwordVariable: 'GITLAB_TOKEN'
                        ),
                        usernamePassword(
                            credentialsId: 'github-token',
                            usernameVariable: 'GITHUB_USERNAME',
                            passwordVariable: 'GITHUB_TOKEN'
                        )
                    ]) {
                        sh "docker login -u ${GITLAB_USERNAME} -p ${GITLAB_TOKEN} registry.gitlab.com"
                        sh """
                            docker build \
                                --build-arg GITHUB_TOKEN=${GITHUB_TOKEN} \
                                --no-cache \
                                -f apps/live/Dockerfile.live \
                                -t ${GITLAB_REP}:${IMAGE_TAG}-live \
                                .
                        """
                        sh "docker push ${GITLAB_REP}:${IMAGE_TAG}-live"
                        sh "docker image rm -f ${GITLAB_REP}:${IMAGE_TAG}-live || true"
                    }
                }
            }
        }

        stage('Cleanup local image') {
            steps {
                script {
                    // Remove dangling layers created during build
                    sh "docker image prune -f || true"
                }
            }
        }
    }

    post {
        success {
            script {
                // Notify on Discord only if the build is successful
                discordSend(
                    webhookUrl: "${DISCORD_WEBHOOK}",
                    message: "${env.REPO_FULLNAME} pipeline successful! :tada:",
                    color: 2276147,
                    successful: true,
                    link: "${env.BUILD_URL}",
                    e2eLink: "${REPORT_LINK}",
                    buildNo: "${env.BUILD_NUMBER}",
                    imageTag: "${IMAGE_TAG}",
                    commitMessage: "${env.GIT_COMMIT_MESSAGE}",
                    commitAuthor: "${env.GIT_COMMIT_AUTHOR}"
                )
            }
        }
        failure {
            script {
                // Notify on Discord only if the build fails
                discordSend(
                    webhookUrl: "${DISCORD_WEBHOOK}",
                    message: "${env.REPO_FULLNAME} pipeline failed! :x:",
                    color: 12263716,
                    successful: false,
                    link: "${env.BUILD_URL}",
                    e2eLink: "${REPORT_LINK}",
                    buildNo: "${env.BUILD_NUMBER}",
                    imageTag: "${IMAGE_TAG}",
                    commitMessage: "${env.GIT_COMMIT_MESSAGE}",
                    commitAuthor: "${env.GIT_COMMIT_AUTHOR}"
                )
            }
        }
    }
}

def discordSend(Map params) {
    def escapedCommitMessage = params.commitMessage?.replaceAll('"', '\\\\"')?.replaceAll('\n', '\\\\n') ?: 'No commit message'
    def payload = """{
        "username": "Jenkins",
        "embeds": [{
            "title": "${params.message}",
            "url": "${params.link}",
            "color": ${params.color},
            "description": "E2E report => [Report](${params.e2eLink})",
            "fields": [
                {"name": "Build No", "value": "${params.buildNo}"},
                {"name": "Image Tag", "value": "${params.imageTag}"},
                {"name": "Commit Message", "value": "${escapedCommitMessage}"},
                {"name": "Author", "value": "${params.commitAuthor ?: 'Unknown'}"}
            ]
        }]
    }"""
    sh "curl -X POST -H 'Content-Type: application/json' -d '${payload}' ${params.webhookUrl}"
}
