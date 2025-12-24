pipeline {
    agent any

    tools {
        nodejs 'NodeJS-18'
        dotnetsdk 'dotnet-sdk-8'
    }

    environment {
        ODC_REPORTS = "${WORKSPACE}/dependency-check-reports"
        SONAR_SCANNER = tool 'SonarScanner'
    }

    stages {

        stage('Validate Target') {
            when {
                changeRequest()
            }
            steps {
                script {
                    echo "PR Target Branch: ${env.CHANGE_TARGET}"
                    if (env.CHANGE_TARGET != 'dev') {
                        error("PR target is not 'dev'. Skipping security scan.")
                    }
                }
            }
        }

        stage('Checkout') {
            when {
                anyOf {
                    branch 'dev'
                    changeRequest()
                }
            }
            steps {
                checkout scm
            }
        }

        stage('Build & Test') {
            when {
                anyOf {
                    branch 'dev'
                    changeRequest()
                }
            }
            steps {
                sh '''
                    npm install --legacy-peer-deps
                    npm run build
                    dotnet restore
                    dotnet build --configuration Release
                '''
            }
        }

        stage('SonarQube Analysis') {
            when {
                anyOf {
                    branch 'dev'
                    changeRequest()
                }
            }
            steps {
                withSonarQubeEnv('SonarQube') {
                    sh """
                        ${SONAR_SCANNER}/bin/sonar-scanner \
                        -Dsonar.projectKey=CodeClash \
                        -Dsonar.projectName=CodeClash \
                        -Dsonar.sources=. \
                        -Dsonar.pullrequest.key=${env.CHANGE_ID} \
                        -Dsonar.pullrequest.branch=${env.CHANGE_BRANCH} \
                        -Dsonar.pullrequest.base=${env.CHANGE_TARGET}
                    """
                }
            }
        }

        stage('Quality Gate') {
            when {
                anyOf {
                    branch 'dev'
                    changeRequest()
                }
            }
            steps {
                timeout(time: 2, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('OWASP Dependency-Check (SCA)') {
            when {
                anyOf {
                    branch 'dev'
                    changeRequest()
                }
            }
            steps {
                dependencyCheck(
                    odcInstallation: 'dependency-check',
                    additionalArguments: """
                        --scan .
                        --project CodeClash
                        --out ${ODC_REPORTS}
                        --format ALL
                        --failOnCVSS 7
                    """
                )
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'dependency-check-reports/**/*', fingerprint: true
            dependencyCheckPublisher pattern: 'dependency-check-reports/dependency-check-report.xml'
        }

        failure {
            echo '❌ PR or dev build blocked due to security violations'
        }
    }
}
