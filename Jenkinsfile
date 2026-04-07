pipeline {
    agent {
        label "generic_arm64"
    }

    tools {
        jfrog 'jfrog-cli'
    }

    parameters {
        string(name: 'BRANCH', defaultValue: 'master', description: "Git branch to use")
        string(name: 'VERSION', defaultValue: '7.4.24', description: "JBoss Version to use")
    }

        environment {
            JFROG_SERVER_ID    = 'sheerid-jfrog'
        }


    stages {

        stage ('Clone') {
            steps {
                echo "Pulling pangaea code for branch ${params.BRANCH}"
                checkout([
                    $class: 'GitSCM',
                    branches: [[name: "${params.BRANCH}"]],
                    userRemoteConfigs: [
                        [url: 'git@github.com:sheerid/eap-build.git',
                         credentialsId: 'ssh-github-sheerid-build']
                    ]
                ])
            }
        }

        stage('Build') {
            steps {
                dir(".") {
                    sh """
                    ./build-eap7.sh ${params.VERSION}
                    """
                }
            }
        }

        stage('Upload to Artifactory') {
            steps {
                dir("dist") {
                    jf 'rt u --server-id=${JFROG_SERVER_ID} *.zip generic-local/jboss/'
                    jf 'rt build-collect-env'
                    jf 'rt build-publish'
                }
            }
        }

        stage('Xray Scan') {
            steps {
                jf """build-scan \
                    --server-id=${JFROG_SERVER_ID} \
                    --vuln \
                    --fail=false"""
            }
        }
    }
}