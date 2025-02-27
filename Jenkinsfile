def sendEmail (job_name, job_id, job_status, tests_output) {
    echo "----------------------------------------------"
    echo "JOB NAME: ${job_name}"
    echo "JOB ID: ${job_id}"
    echo "JOB STATUS: ${job_status}"
    echo "----------------------------------------------"
    echo "TESTS OUTPUT:"
    echo "${tests_output}"
}

def output = ""

pipeline {
    agent any
    parameters {
        string defaultValue: 'DPO', name: 'ARTIFACT_NAME', trim: true
        booleanParam 'FAIL_PIPELINE'
        booleanParam defaultValue: true, name: 'RUN_TEST'
    }

    stages {
        stage('Clean') {
            steps {
                cleanWs()
            }
        }
        stage('Download') {
            steps {
                echo (message: "Download")
                rtDownload (
                    serverId: 'acrtifactory-jenkins-course',
                    spec: '''{
                        "files": [
                            {
                                "pattern" : "generic-local/libraries/printer.zip",
                                "target"  : "generic-local/libraries/"
                            }
                        ]
                    }'''
                )
                dir('pipeline') {
                    git (
                            url: 'https://github.com/KLevon/jenkins-course', 
                            branch: 'pipeline'
                    )
                }
                unzip (
                    zipFile: "generic-local/libraries/libraries/printer.zip",
                    dir: "pipeline/"
                )
            }
        }
        stage('Build') {
            steps {
                echo (message: "Build")
                dir ('pipeline') {
                    bat (
                        script: "Makefile.bat"    
                    )
                }
                
                withCredentials (
                    [usernamePassword(credentialsId: '123', passwordVariable: 'pwd', usernameVariable: 'usr')]    
                ) {
                    echo (message: "Credentials [Username/Password]: $usr / $pwd")
                }
            }
        }
        stage('Tests') {
            when {
                equals expected: true,
                actual: params.RUN_TEST
            }
            steps {
                dir ('pipeline') {
                    script {
                        def modules = ["printer", "scanner", "main"]
                    
                        for (module in modules) {
                            output += bat (script: "Tests.bat ${module}", returnStdout: true).trim() + "\r\n"
                        }
                    }
                }
            }
        }
        stage('Publish') {
            steps {
                echo (message: "Publish")
                script {
                    zip (
                        zipFile: "pipeline.zip",
                        archive: true,
                        dir: "pipeline"
                    )
                }
                rtUpload (
                    serverId: 'acrtifactory-jenkins-course',
                    spec: """{
                        "files": [
                            {
                                "pattern" : "pipeline.zip",
                                "target"  : "generic-local/release/dusko/${env.BUILD_ID}/pipeline-${params.ARTIFACT_NAME}.zip"
                            }
                        ]
                    }"""
                )
                
                script {
                    if (params.FAIL_PIPELINE == true) {
                        bat (
                            script: "exit 1"
                        )
                    }
                }
            }
        }
    }
    
    post {
        failure {
            script {
                sendEmail (env.JOB_NAME, env.BUILD_ID, currentBuild.currentResult, output)
            }
        }
    }
}