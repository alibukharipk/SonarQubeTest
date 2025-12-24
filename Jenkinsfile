pipeline {
    agent any

    tools {
        nodejs 'NodeJS-18'
    }

    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
        SONAR_SCANNER = tool 'SonarScanner'
    }

    stages {

        stage('Checkout') {
            steps {
                checkout scm
            }
        }

        stage('Build Backend (.NET 8)') {
            steps {
                dir('backend') {
                    sh '''
                      dotnet restore
                      dotnet build --no-restore
                    '''
                }
            }
        }

        stage('Build Frontend (React)') {
            steps {
                dir('frontend') {
                    sh '''
                      npm install --legacy-peer-deps
                      npm run build
                    '''
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                      dotnet ${SONAR_SCANNER}/SonarScanner.MSBuild.dll begin \
                        /k:"github-dotnet-react" \
                        /n:"GitHub .NET + React" \
                        /d:sonar.host.url=$SONAR_HOST_URL \
                        /d:sonar.login=$SONAR_AUTH_TOKEN \
                        /d:sonar.sources=SonarTestApp.Server,sonartestapp.client \
                        /d:sonar.exclusions=**/node_modules/**,**/build/**

                      dotnet build backend

                      dotnet ${SONAR_SCANNER}/SonarScanner.MSBuild.dll end \
                        /d:sonar.login=$SONAR_AUTH_TOKEN
                    """
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
    }
}
