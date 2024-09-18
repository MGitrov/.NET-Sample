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
                    Write-Host "Deploy Path: ${env:DEPLOY_PATH}"
                    Write-Host "Web App Pool: ${env:WEB_APP_POOL}"
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
                    echo "Restoring dependencies for ${env.SOLUTION_FILE}..."

                    // Pulls packages from the "NuGet.Config" file sources.
                    powershell "dotnet restore ${env:SOLUTION_FILE}" // --source ${env:NUGET_SOURCE}
                }
            }
        }

        stage("Build solution file") {
            steps {
                echo "Building ${env.SOLUTION_FILE}..."

                // Builds all the projects contained in the provided solution file.
                powershell "dotnet build ${env:SOLUTION_FILE}"
            }
        }

        stage("Prepare for deployment (a.k.a publish)") {
            steps {
                echo "Publishing ${env.SOLUTION_FILE}..."

                // Compiles the application and prepare it for deployment.
                powershell "dotnet publish ${env:SOLUTION_FILE} -c ${env:CONFIGURATION} -r ${env:TARGET_RUNTIME} -o ${env:OUTPUT_DIRECTORY} --self-contained true"
            }
        }

        stage("Deployment to IIS web server") {
            steps {
                echo "Copying application files from ${env.OUTPUT_DIRECTORY} to ${env.DEPLOY_PATH}..."

                powershell "Copy-Item -Recurse -Force ${env:OUTPUT_DIRECTORY}\\* '${env:DEPLOY_PATH}'"
            }
        }

        stage("Recycle web app pool") {
            steps {
                echo "Recycling ${env:WEB_APP_POOL} web app pool..."

                powershell 'Restart-WebAppPool -Name "${env:WEB_APP_POOL}"'
            }
        }
    }
    
    post {
        always {
            cleanWs()
        }
    }
}