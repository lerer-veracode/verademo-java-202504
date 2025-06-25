//
// This is an example of using VeraDemo Java test application with the Veracode Static scanner.  
//

pipeline {
    agent any

    environment {
        VERACODE_APP_NAME = 'Verademo Java'      // App Name in the Veracode Platform
    }

    stages{
        stage ('environment verify') {
            steps {
                script {
                    if (isUnix() == true) {
                        sh 'pwd'
                        sh 'ls -la'
                        sh 'echo $PATH'
                        sh 'echo $M2_HOME'
                    }
                    else {
                        bat 'dir'
                        bat 'echo %PATH%'
                    }
                }
            }
        }

        stage ('build') {
            steps {
                script {
                    if(isUnix() == true) {
                        // Compile Java app
                        withMaven (traceability: true, maven: '3.9.10') {
                            sh script:'''
                              #!/bin/bash
                              cd ./app
                              ls -la
                              mvn clean package
                            '''
                        }
                        //sh 'mvn -f app clean package'
                    }
                    //else {
                    //    bat 'mvn clean package'
                        //bat 'mvn -f app clean package'
                    //}
                }
            }
        }
        stage('Veracode AST') {
            parallel {
                stage ('Veracode Scan') {
                    steps {
                        script {
                            if(isUnix() == true) {
                                env.HOST_OS = 'Unix'
                            }
                            else {
                                env.HOST_OS = 'Windows'
                            }
                        }
                        echo 'Veracode scanning'
                        withCredentials([usernamePassword(credentialsId: 'Veracode-API-credentials', passwordVariable: 'veracode_key', usernameVariable: 'veracode_id')]) {
                            veracode applicationName: 'Verademo Java', createSandbox: true, criticality: 'High', debug: true, deleteIncompleteScanLevel: '1', fileNamePattern: '', includenewmodules: true, replacementPattern: '', sandboxName: 'Jenkins', scanExcludesPattern: '', scanIncludesPattern: '', scanName: 'build $buildnumber - Jenkins', scanallnonfataltoplevelmodules: true, teams: '', uploadIncludesPattern: '**/target/**.zip,**/target/*.war', vid: veracode_id, vkey: veracode_key
                        }
                        // Add timeout: X to the param list for waif for completion
                    }
                }

                stage ('Veracode SCA') {
                    steps {
                        echo 'Veracode SCA'
                        withCredentials([ string(credentialsId: 'SRCCLR_API_TOKEN', variable: 'SRCCLR_API_TOKEN')]) {
                            //withMaven(maven:'maven-3') {
                                script {
                                    if(isUnix() == true) {
                                        sh "curl -sSL https://download.sourceclear.com/ci.sh | sh -s -- scan app"
        
                                        // debug, no upload
                                        //sh "curl -sSL https://download.sourceclear.com/ci.sh | DEBUG=1 sh -s -- scan --no-upload"
                                    }
                                    else {
                                        powershell '''
                                                    Set-ExecutionPolicy AllSigned -Scope Process -Force
                                                    $ProgressPreference = "silentlyContinue"
                                                    iex ((New-Object System.Net.WebClient).DownloadString('https://download.srcclr.com/ci.ps1'))
                                                    srcclr scan app
                                                    '''
                                    }
                                }
                            //}
                        }
                    }
                }

                stage ('Veracode Pipeline Scan') {
                    steps {
                        //Pipeline scan
                        catchError {
                            withCredentials([usernamePassword(credentialsId: 'Veracode-API-credentials', passwordVariable: 'veracode_key', usernameVariable: 'veracode_id')]) {
                                sh 'curl -O https://downloads.veracode.com/securityscan/pipeline-scan-LATEST.zip'
                                sh 'unzip -o pipeline-scan-LATEST.zip pipeline-scan.jar'
                                sh '''java -jar pipeline-scan.jar -vid "$veracode_id" -vkey "$veracode_key" --file app/target/verademo.war -sf pipeline_output.txt -so true'''
                            }
                        }
                        archiveArtifacts artifacts: 'pipeline_output.txt', followSymlinks: false
                        
                    }
                }

                // Currently only works on *nix
                stage ('Veracode container scan') {
                    steps {
                        echo 'Veracode container scanning'
                        withCredentials([usernamePassword(
                            credentialsId: 'Veracode-API-credentials', passwordVariable: 'veracode_key', usernameVariable: 'veracode_id')]) {
                                script {
                                    if(isUnix() == true) {
                                        sh '''
                                            curl -fsS https://tools.veracode.com/veracode-cli/install | sh
                                            ./veracode scan --type directory --source . --format table
                                            '''
                                    }
                                }
                            }
                    }
                }
            }
        }
    }
}
