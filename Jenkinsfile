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
                    Write-Host "Code Repository URL: ${env:CODE_REPOSITORY}"
                    Write-Host "Main Branch: ${env:MAIN_BRANCH}"
                    Write-Host "Frontend Path: ${env:FRONTEND_PATH}"
                    Write-Host "Frontend Configuration: ${env:FRONTEND_CONFIGURATION}"
                    Write-Host "Frontend Base Href: ${env:FRONTEND_BASE_HREF}"
                    Write-Host "Frontend Output Directory: ${env:FRONTEND_OUTPUT_DIRECTORY}"
                    Write-Host "Backend Path: ${env:BACKEND_PATH}"
                    Write-Host "Backend NuGet Repository URL: ${env:BACKEND_NUGET_SOURCE}"
                    Write-Host "Backend Solution File: ${env:BACKEND_SOLUTION_FILE}"
                    Write-Host "Backend Configuration: ${env:BACKEND_CONFIGURATION}"
                    Write-Host "Backend Target Runtime: ${env:BACKEND_TARGET_RUNTIME}"
                    Write-Host "Backend Output Directory: ${env:BACKEND_OUTPUT_DIRECTORY}"
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
                              userRemoteConfigs: [[url: "${env:CODE_REPOSITORY}"]]
                    ])

                    powershell "ls" // Ensures that Jenkins pulled all the files.
                }
            }
        }

        /*stage("Build Frontend (Angular)") {
            steps {
                script {
                    dir(FRONTEND_PATH) {
                        echo "Installing dependencies..."
                        powershell "npm install"

                        echo "Building Angular frontend..."
                        powershell "ng build --configuration ${env:FRONTEND_CONFIGURATION} --base-href=${env:FRONTEND_BASE_HREF}"
                    }
                }
            }
        }*/

        stage("Restore dependencies") {
            steps {
                script {
                    echo "Restoring dependencies for ${env.BACKEND_SOLUTION_FILE}..."

                    // Pulls packages from the "NuGet.Config" file sources.
                    powershell "dotnet restore ${env:BACKEND_SOLUTION_FILE}" // --source ${env:NUGET_SOURCE}
                }
            }
        }

        stage("Build solution file") {
            steps {
                echo "Building ${env.BACKEND_SOLUTION_FILE}..."

                // Builds all the projects contained in the provided solution file.
                powershell "dotnet build ${env:BACKEND_SOLUTION_FILE}"
            }
        }

        stage("Prepare for deployment (a.k.a publish)") {
            steps {
                echo "Publishing ${env.BACKEND_SOLUTION_FILE}..."

                // Compiles the application and prepare it for deployment.
                powershell "dotnet publish ${env:BACKEND_SOLUTION_FILE} -c ${env:BACKEND_CONFIGURATION} -r ${env:BACKEND_TARGET_RUNTIME} -o ${env:BACKEND_OUTPUT_DIRECTORY} --self-contained true"
            }
        }

        stage("Deployment to IIS web server") {
            steps {
                echo "Deploying backend..."
                echo "Copying application files from ${env.BACKEND_OUTPUT_DIRECTORY} to ${env.DEPLOY_PATH}..."

                powershell "Copy-Item -Recurse -Force ${env:BACKEND_OUTPUT_DIRECTORY}\\* '${env:DEPLOY_PATH}'"

                /*echo "Deploying frontend..."
                echo "Copying application files from ${env.FRONTEND_OUTPUT_DIRECTORY} to ${env.DEPLOY_PATH}...""*/

                //powershell "Copy-Item -Recurse -Force ${env:FRONTEND_PATH}\\${env:FRONTEND_OUTPUT_DIRECTORY}\\* '${env:DEPLOY_PATH}'"
            }
        }

        stage("Recycle web app pool") {
            steps {
                echo "Recycling ${env:WEB_APP_POOL} web app pool..."

                powershell "Restart-WebAppPool -Name '${env:WEB_APP_POOL}'"
            }
        }
    }

    post {
        always {
            cleanWs()
        }
    }
}