pipeline {
    agent {label "Local-Agent"}

    stages {
        stage("Load environmet variables") {
            steps {
                script {
                    echo "Loading environment variables from the '.env' file..."

                    // Reads the ".env" file.
                    def envVariables = powershell(returnStdout: true, script: "Get-Content .env -Raw")
                                    
                    // Parses the contents of the ".env" file, and sets the environment variables in the pipeline.
                    envVariables.split("\r?\n").each { line ->
                    def keyValue = line.split("=", 2)
                    if (keyValue.size() == 2) {
                        def key = keyValue[0].trim()
                        def value = keyValue[1].trim()
                        env."${key}" = value
                        echo "Setting ${key} to ${value}"
                        }
                    }
                }
            }
        }

        stage("Verify environment variables") {
            steps {
                script {
                    // Jenkins parameters are available to PowerShell as an environment variable.
                    powershell '''
                    Write-Host "NuGet Repository URL: ${env:NUGET_SOURCE}"
                    Write-Host "Main Branch: ${env:MAIN_BRANCH}"
                    Write-Host "Solution File: ${env:SOLUTION_FILE}"
                    Write-Host "Configuration: ${env:CONFIGURATION}"
                    Write-Host "Target Runtime: ${env:TARGET_RUNTIME}"
                    Write-Host "Output Directory: ${env:OUTPUT_DIRECTORY}"
                    '''
                }
            }
        }

        stage("Checkout the main branch") {
            steps {
                script {
                    echo "Building ${env.MAIN_BRANCH} branch..."

                    checkout([$class: "GitSCM", branches: [[name: "*/${env.MAIN_BRANCH}"]],
                              userRemoteConfigs: [[url: "${env:NUGET_SOURCE}"]]
                    ])

                    powershell "ls" // Ensures that Jenkins pulled all the files.
                }
            }
        }

        stage("Restore dependencies") {
            steps {
                script {
                    // Pulls packages from the "NuGet.Config" file sources.
                    powershell "dotnet restore ${env:SOLUTION_FILE}" // --source ${env:NUGET_SOURCE}
                }
            }
        }

        /*stage("Deployment package creation") {
            steps {
                    script {
                        deploymentPackageCreation()
                    }
            }
        }*/

        /*stage("Verify ZIP file creation") {
            steps {
                powershell '''
                if (Test-Path "$env:WORKSPACE\\$env:PACKAGE_NAME") {
                    Write-Host "ZIP file created successfully: $env:PACKAGE_NAME"
                    ls
                } else {
                    Write-Error "ZIP file was not created"
                }
                '''
            }
        }*/

        /*stage("ZIP file contents verification") {
            steps {
                echo "Verifying contents of the ZIP file..."
                powershell '''
                Add-Type -AssemblyName System.IO.Compression.FileSystem
                $zipFile = [System.IO.Compression.ZipFile]::OpenRead("$env:WORKSPACE\\$env:PACKAGE_NAME")
                $zipFile.Entries | ForEach-Object { $_.FullName }
                $zipFile.Dispose()
                '''
            }
        }*/

        /*stage("Deployment to IIS web server") {
            steps {
                script {
                    deploymentToIIS()
                }
            }
        }*/

        /*stage("Recycling web app pool") {
            steps {
                script {
                    recycleWebAppPool()
                }
            }
        }*/
    }
    post {
        always {
            cleanWs()
        }
    }
}