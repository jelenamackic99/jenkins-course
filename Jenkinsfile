def output = ""
def sendEmail(jobName, jobId, output)
    {
        echo "I am sending email $jobName and $jobId   $output" 
    }
pipeline {
    agent any
    parameters {
              string defaultValue: 'pipeline.zip', description:'File name', name: 'ARTIFACT_NAME'
              booleanParam defaultValue: true, description: 'Check if build needs to fail', name: 'FAIL_PIPELINE'
    }

    stages{
        stage('Download'){
            steps{
                cleanWs()
                dir('jelena')
                {
                    git (
                        branch: "$params.GIT_BRANCH" ,
                        url: 'https://github.com/KLevon/jenkins-course.git'
                    )
                }
                rtDownload(
                    serverId: 'Artifactory',
                    spec: ''' {
                        "files": [
                            {
                                "pattern": "generic-local/libraries/printer.zip",
                                "target": "./jelena/",
                                "flat": "true"
                            }
                        
                        ]
                    }'''
                )
                unzip(
                    zipFile: "./jelena/printer.zip",
                    dir: "./jelena/"
                )
                
                
            }
        }
         
        stage('Build'){
            steps{
                bat (
                    script: """
                        cd jelena/
                        ./Makefile.bat
                    """
                )
                echo(message: "ISCon 2025")
            }
        }
        stage('Test'){
            when {
                equals expected: true,
                actual: params.RUN_TEST
            }
            steps{
                script{
                    def array = ['printer', 'scanner', 'main']
                    
                    for(element in array)
                    {
                        output += bat (script: "./jelena/Tests.bat $element", returnStdout: true).trim()
                    }
                }
            }
        }
        stage('Publish'){
            steps{
                script {
                    zip(
                        zipFile:"jelena.zip",
                        archive: true,
                        dir: "./jelena"
                    )
                }
                
                rtUpload(
                    serverId: 'Artifactory',
                    spec: """ {
                        "files": [
                            {
                                "pattern": "./jelena.zip",
                                "target": "generic-local/jelena/${env.BUILD_ID}/${params.ARTIFACT_NAME}/"
                            }
                        
                        ]
                    }"""
                )
                
                echo(message: "ISCon 2025")
                
                script {

                    if (params.FAIL_PIPELINE)

                    {

                        but "exit 1";

                    }
                    
                    
                }
 
            }
        }
        
    }
    post {
        failure {
            script{
                if(params.SEND_EMAIL)
                {
                    sendEmail(env.JOB_NAME, env.BUILD_ID, output)
                }
            }
        }
    }
    
}
