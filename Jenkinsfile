@Library('ecommerce-shared-lib') _

def SERVICE_NAME    = 'order-service'
def DOCKER_REGISTRY = 'yeelaine'
def GIT_SHORT       = ''
def IMAGE_TAG       = ''
def FULL_IMAGE      = ''

pipeline {

    agent any

    environment {
        DOCKER_BUILDKIT = '1'
    }

    options {
        timestamps()
        timeout(time: 30, unit: 'MINUTES')
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
                script {
                    GIT_SHORT  = sh(script: 'git rev-parse --short HEAD', returnStdout: true).trim()
                    IMAGE_TAG  = "git-${GIT_SHORT}"
                    FULL_IMAGE = "${DOCKER_REGISTRY}/ecommerce-${SERVICE_NAME}:${IMAGE_TAG}"
                    echo "Image will be tagged: ${FULL_IMAGE}"
                }
            }
        }

        stage('Build') {
            steps {
                buildStage(service: SERVICE_NAME)
            }
        }

        stage('Test') {
            steps {
                testStage(service: SERVICE_NAME)
            }
        }

        stage('Security Scan - IaC (Checkov)') {
            steps {
                securityScanStage(service: SERVICE_NAME)
            }
        }

        stage('Docker Build & Push') {
            when { not { changeRequest() } }
            steps {
                script {
                    dockerBuildPush(service: SERVICE_NAME, tag: IMAGE_TAG, registry: DOCKER_REGISTRY)
                    FULL_IMAGE = env.DOCKER_IMAGE
                }
            }
        }

        stage('Security Scan - Image (Trivy)') {
            when { not { changeRequest() } }
            steps {
                securityScanStage(service: SERVICE_NAME, image: FULL_IMAGE)
            }
        }

        stage('Deploy to Dev') {
            when { branch 'develop' }
            steps {
                k8sDeploy(service: SERVICE_NAME, namespace: 'dev', image: FULL_IMAGE)
            }
        }

        stage('Deploy to Staging') {
            when { expression { env.BRANCH_NAME ==~ /release\/.*/ } }
            steps {
                k8sDeploy(service: SERVICE_NAME, namespace: 'staging', image: FULL_IMAGE)
            }
        }

        stage('Approval Gate — Production') {
            when { branch 'main' }
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    input message: "Deploy ${FULL_IMAGE} to PRODUCTION?",
                          ok: 'Approve',
                          submitter: 'admin'
                }
            }
        }

        stage('Deploy to Production') {
            when { branch 'main' }
            steps {
                k8sDeploy(service: SERVICE_NAME, namespace: 'prod', image: FULL_IMAGE)
            }
        }

        stage('Update Monitoring Stack') {
            when { branch 'main' }
            steps {
                monitoringDeploy(namespace: 'monitoring')
            }
        }

    }

    post {
        success { echo "SUCCESS — ${SERVICE_NAME} @ ${IMAGE_TAG}" }
        failure { echo "FAILED — ${SERVICE_NAME} @ ${IMAGE_TAG}" }
        always  { cleanWs() }
    }

}
