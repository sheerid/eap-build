pipeline {
    agent any

    parameters {
        string(name: 'branch', defaultValue: 'master', description: "Git branch to use")
        string(name: 'version', defaultValue: '6.4.17', description: "JBoss Version to use")
    }

    stages {

      stage('Clone Repos') {
          steps {
              git url: "git@github.com:sheerid/eap-build", branch: "${params.branch}"
          }
      }

        stage('Build') {
            steps {
                dir(".") {
                    sh """
                    ./build-eap7.sh ${params.version}
                    """
                }
            }
        }

        stage('Deploy') {
            steps {
                dir(".") {
                    script {
                        def server = Artifactory.server('Artifactory')
                        def uploadSpec = """{
                            "files": [
                                {
                                    "pattern": "dist/*.zip",
                                    "target": "generic-local/jboss/"
                                }
                                ]
                            }"""
                        def buildInfo = server.upload(uploadSpec)
                        server.publishBuildInfo buildInfo
                    }
                }
            }
        }
    }
}