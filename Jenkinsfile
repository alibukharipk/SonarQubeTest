pipeline {
    agent any

    tools {
        nodejs 'NodeJS-22'
    }

    environment {
        DOTNET_CLI_TELEMETRY_OPTOUT = '1'
        DOTNET_NOLOGO = '1'
    }

    stages {

        /* =======================
           REACTJS BUILD
        ======================= */
        stage('ReactJS - Install & Build') {
            steps {
                dir('sonartestapp.client') {
                    sh '''
                      node -v
                      npm -v
                      npm install --legacy-peer-deps
                      npm run build
                    '''
                }
            }
        }

        /* =======================
           .NET 8 BUILD + SECURITYCODESCAN
        ======================= */
        stage('.NET 8 - Restore & Build') {
            steps {
                dir('SonarTestApp.Server') {
                    sh '''
                      dotnet restore
                      dotnet build \
                        --configuration Release \
                        /warnaserror \
                        /p:RunAnalyzers=true \
                        /p:EnableNETAnalyzers=true \
                        /p:AnalysisLevel=latest
                    '''
                }
            }
        }

        /* =======================
           SONARQUBE ANALYSIS
        ======================= */
        stage('SonarQube Analysis') {
            steps {
                script {
                    def scannerHome = tool 'SonarScanner'
                    withSonarQubeEnv('SonarQube') {
                        sh """
                        ${scannerHome}/bin/sonar-scanner \
                          -Dsonar.projectKey=ReactJS-SonarTest \
                          -Dsonar.projectName=ReactJS-SonarTest \
                          -Dsonar.sources=sonartestapp.client/src,SonarTestApp.Server \
                          -Dsonar.javascript.lcov.reportPaths=sonartestapp.client/coverage/lcov.info
                        """
                    }
                }
            }
        }

        /* =======================
           SONARQUBE QUALITY GATE
        ======================= */
        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 5, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        /* =======================
           OWASP DEPENDENCY-CHECK
        ======================= */
        stage('OWASP Dependency-Check') {
            steps {
                dependencyCheck additionalArguments: '''
                    --scan .
                    --format ALL
                    --failOnCVSS 7
                    --project "ReactJS-SonarTest"
                ''',
                odcInstallation: 'dependency-check'
            }
        }
    }

    post {
        always {
            dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
        }

        success {
            echo '✅ Build PASSED – No critical security issues found'
        }

        failure {
            echo '❌ Build FAILED – Critical security issues detected'
        }
    }
}
