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
           SONARQUBE QUALITY GATE
        ======================= */
        stage('SonarQube Quality Gate') {
            steps {
                timeout(time: 15, unit: 'MINUTES') {
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
                    --project "SonarTestApp (React + .NET)"
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
