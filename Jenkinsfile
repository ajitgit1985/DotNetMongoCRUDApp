pipeline {
    agent any

    environment {
        APP_NAME = 'DotNetMongoCRUDApp'
        APP_POOL = 'DotNetMongoCRUDAppPool'
        DEPLOY_DIR = 'C:\\Deploy\\DotNetMongoCRUDApp'
        PUBLISH_DIR = 'C:\\JenkinsPublish\\DotNetMongoCRUDApp'
        PROJECT_FILE = 'DotNetMongoCRUDApp.csproj'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Restore') {
            steps {
                bat 'dotnet restore'
            }
        }

        stage('Build') {
            steps {
                bat 'dotnet build %PROJECT_FILE% -c Release --no-restore'
            }
        }

        stage('Publish') {
            steps {
                bat 'if exist "%PUBLISH_DIR%" rmdir /S /Q "%PUBLISH_DIR%"'
                bat 'dotnet publish %PROJECT_FILE% -c Release --no-build -o "%PUBLISH_DIR%"'
            }
        }

        stage('Stop IIS') {
            steps {
                powershell '''
                    Import-Module WebAdministration

                    Stop-WebAppPool -Name "$env:APP_POOL"

                    Start-Sleep -Seconds 3
                '''
            }
        }

        stage('Deploy') {
            steps {
                powershell '''
                    robocopy "$env:PUBLISH_DIR" "$env:DEPLOY_DIR" /MIR

                    if ($LASTEXITCODE -le 7) {
                        exit 0
                    }

                    exit $LASTEXITCODE
                '''
            }
        }

        stage('Start IIS') {
            steps {
                powershell '''
                    Import-Module WebAdministration

                    Start-WebAppPool -Name "$env:APP_POOL"

                    Start-Sleep -Seconds 3

                    Start-Website -Name "$env:APP_NAME"
                '''
            }
        }
    }
}