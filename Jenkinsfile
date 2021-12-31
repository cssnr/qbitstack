#!/usr/bin/env groovy

@Library('jenkins-libraries')_

pipeline {
    agent {
        label 'manager'
    }
    options {
        buildDiscarder(logRotator(numToKeepStr:'5'))
        timeout(time: 1, unit: 'HOURS')
    }
    environment {
        DISCORD_ID   = "smashed-alerts"
        COMPOSE_FILE = "docker-compose-swarm.yml"

        BUILD_CAUSE = getBuildCause()
        VERSION = getVersion("${GIT_BRANCH}")
        GIT_ORG = getGitGroup("${GIT_URL}")
        GIT_REPO = getGitRepo("${GIT_URL}")
        BASE_NAME = "${GIT_ORG}-${GIT_REPO}"
        SERVICE_NAME = "${BASE_NAME}"
        NFS_HOST = "nfs01.cssnr.com"
        NFS_BASE = "/data/docker"
    }
    stages {
        stage('Init') {
            steps {
                echo "\n--- Build Details ---\n" +
                        "GIT_URL:       ${GIT_URL}\n" +
                        "JOB_NAME:      ${JOB_NAME}\n" +
                        "SERVICE_NAME:  ${SERVICE_NAME}\n" +
                        "NFS_HOST:      ${NFS_HOST}\n" +
                        "NFS_BASE:      ${NFS_BASE}\n" +
                        "BASE_NAME:     ${BASE_NAME}\n" +
                        "BUILD_CAUSE:   ${BUILD_CAUSE}\n" +
                        "GIT_BRANCH:    ${GIT_BRANCH}\n" +
                        "VERSION:       ${VERSION}\n"
                verifyBuild()
                sendDiscord("${DISCORD_ID}", "Pipeline Started by: ${BUILD_CAUSE}")
            }
        }
        stage('Dev Deploy') {
            when {
                allOf {
                    not { branch 'master' }
                }
            }
            environment {
                STACK_NAME = "dev_${BASE_NAME}"
                NFS_DIRECTORY = "${NFS_BASE}/${STACK_NAME}"
                TRAEFIK_HOST = "qbittorrent-dev.cssnr.com"
                QBIT_PORT = "40103"
            }
            steps {
                echo "\n--- Starting Dev Deploy ---\n" +
                        "STACK_NAME:    ${STACK_NAME}\n" +
                        "QBIT_PORT:     ${QBIT_PORT}\n" +
                        "NFS_HOST:      ${NFS_HOST}\n" +
                        "NFS_DIRECTORY: ${NFS_DIRECTORY}\n"
                sendDiscord("${DISCORD_ID}", "Dev Deploy Started")
                echo "Setting up NFS"
                setupNfs("${STACK_NAME}")
                echo "Stack Push"
                updateCompose("${COMPOSE_FILE}", "STACK_NAME", "${STACK_NAME}")
                stackPush("${COMPOSE_FILE}")
                echo "Stack Deploy"
                stackDeploy("${COMPOSE_FILE}", "${STACK_NAME}")
                sendDiscord("${DISCORD_ID}", "Dev Deploy Finished")
            }
        }
        stage('Prod Deploy') {
            when {
                allOf {
                    branch 'master'
                    triggeredBy 'UserIdCause'
                }
            }
            environment {
                STACK_NAME = "prod_${BASE_NAME}"
                NFS_DIRECTORY = "${NFS_BASE}/${STACK_NAME}"
                TRAEFIK_HOST = "qbittorrent.cssnr.com"
                QBIT_PORT = "40104"
            }
            steps {
                echo "\n--- Starting Prod Deploy ---\n" +
                        "STACK_NAME:    ${STACK_NAME}\n" +
                        "QBIT_PORT:     ${QBIT_PORT}\n" +
                        "NFS_HOST:      ${NFS_HOST}\n" +
                        "NFS_DIRECTORY: ${NFS_DIRECTORY}\n"
                sendDiscord("${DISCORD_ID}", "Prod Deploy Started")
                echo "Setting up NFS"
                setupNfs("${STACK_NAME}")
                echo "Stack Push"
                updateCompose("${COMPOSE_FILE}", "STACK_NAME", "${STACK_NAME}")
                stackPush("${COMPOSE_FILE}")
                echo "Stack Deploy"
                stackDeploy("${COMPOSE_FILE}", "${STACK_NAME}")
                sendDiscord("${DISCORD_ID}", "Prod Deploy Finished")
            }
        }
    }
    post {
        always {
            cleanWs()
            script { if (!env.INVALID_BUILD) {
                sendDiscord("${DISCORD_ID}", "Pipeline Complete: ${currentBuild.currentResult}")
            } }
        }
    }
}
