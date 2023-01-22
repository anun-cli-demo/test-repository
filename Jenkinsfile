API_KEY_CREDENTIAL_ID = "anun-dev-e2etenant-webhook-secret"
ANUN_CLI_URL = "https://api.anun-dev.cloud/api/runner/external/get-link"
VERIFY_SSL = false
BUILD_PLATFORM = "jenkins"

INSTANCES = [
  [ANUN_PLATFORM: "github", CREDENTIAL_ID: "anunDev"]
]

pipeline {
    agent { label 'python39' }
    stages {
        stage('Install Anun CLI') {
            steps {
                script {
                    withCredentials([usernamePassword(credentialsId: API_KEY_CREDENTIAL_ID, passwordVariable: 'ANUN_API_KEY', usernameVariable: 'ANUN_TENANT_ID')]) {
                        final String anunCliUrlCreator = "${ANUN_CLI_URL}?tenantId=${ANUN_TENANT_ID}&secret=${ANUN_API_KEY}"
                        final String signedUrlResponse = sh(script: "curl \"$anunCliUrlCreator\"", returnStdout: true)
                        final String anunCliDownloadUrl = new JsonSlurper().parseText(signedUrlResponse).body
                        println("Installing anun-cli...")
                        sh "pip3 install --user \"$anunCliDownloadUrl\""
                    }
                }
            }
        }
        stage('Run Anun CLI') {
            script {
                    withCredentials([usernamePassword(credentialsId: API_KEY_CREDENTIAL_ID, passwordVariable: 'ANUN_API_KEY', usernameVariable: 'ANUN_TENANT_ID')]) {
                        final String pythonUserBase = sh(script: "python3 -m site --user-base", returnStdout: true).trim()
                        final String anunCli = sh(script: "find ${pythonUserBase} -name anun-cli", returnStdout: true).trim()
                        INSTANCES.each { instance ->
                            withCredentials([usernamePassword(credentialsId: instance['CREDENTIAL_ID'], passwordVariable: 'ANUN_TOOL_TOKEN', usernameVariable: 'ANUN_USER')]) {
                                withEnv(instance.collect { param, arg -> "$param=$arg" }) {
                                    println("Invoking anun-cli...")
                                    String verifySSL = VERIFY_SSL ? "--verify-ssl" : ""
                                    sh "$anunCli scan-in-build --build-platform $BUILD_PLATFORM $verifySSL"
                                }
                            }
                        }
                    }
                }
        }
        stage('Build') {
            steps {
                sh 'printenv'
            }
        }
    }
}
