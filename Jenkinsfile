
pipeline {
    agent any

    environment {
        DAST_NETWORK = "dast-pipeline-${BUILD_NUMBER}"
        REPORT_VOLUME = "dast-reports-${BUILD_NUMBER}"
        JUICE_SHOP = "juice-shop-${BUILD_NUMBER}"
    }

    stages {

        stage('Initialisation') {
            steps {
                echo 'Starting DevSecOps Capstone Pipeline'
                echo 'Preparing Jenkins environment'
            }
        }

        stage('Environment Check') {
            steps {
                sh '''
                    echo "Checking Jenkins environment..."
                    java -version
                    git --version
                    docker --version
                '''
            }
        }

        stage('Repository Verification') {
            steps {
                sh '''
                    echo "Checking project repository..."
                    pwd
                    ls -la
                '''
            }
        }

        stage('Deploy DAST Test Environment') {
            steps {
                sh '''
                    set -e

                    docker network create "$DAST_NETWORK"
                    docker volume create "$REPORT_VOLUME"

                    docker run -d \
                        --name "$JUICE_SHOP" \
                        --network "$DAST_NETWORK" \
                        bkimminich/juice-shop

                    echo "Waiting for Juice Shop to start..."
                    sleep 30

                    docker ps --filter "name=$JUICE_SHOP"
                '''
            }
        }

        stage('DAST Scanning') {
            steps {
                script {
                    sh """
                        docker run --rm \
                            -v ${REPORT_VOLUME}:/zap/wrk \
                            alpine chmod 777 /zap/wrk
                    """

                    def zapExit = sh(
                        script: """
                            docker run --rm \
                                --network ${DAST_NETWORK} \
                                -v ${REPORT_VOLUME}:/zap/wrk:rw \
                                zaproxy/zap-stable \
                                zap-baseline.py \
                                -t http://${JUICE_SHOP}:3000 \
                                -r zap-juice-shop-report.html \
                                -J zap-results.json
                        """,
                        returnStatus: true
                    )

                    echo "ZAP exit code: ${zapExit}"

                    if (zapExit == 3) {
                        error('ZAP encountered an execution error.')
                    }

                    if (zapExit < 0 || zapExit > 3) {
                        error("Unexpected ZAP exit code: ${zapExit}")
                    }

                    echo 'ZAP scan completed. Proceeding to report evaluation.'
                }
            }
        }

        stage('Process DAST Results') {
            steps {
                script {
                    sh '''
                        mkdir -p dast-reports

                        docker run --rm \
                            -v "$REPORT_VOLUME":/source \
                            alpine tar -C /source -cf - . \
                            > dast-reports.tar

                        tar -xf dast-reports.tar -C dast-reports
                        rm -f dast-reports.tar

                        echo "Generated reports:"
                        find dast-reports -type f
                    '''

                    archiveArtifacts(
                        artifacts: 'dast-reports/**',
                        allowEmptyArchive: false
                    )

                    if (fileExists('dast-reports/zap-juice-shop-report.html')) {
                        publishHTML([
                            allowMissing: false,
                            alwaysLinkToLastBuild: true,
                            keepAll: true,
                            reportDir: 'dast-reports',
                            reportFiles: 'zap-juice-shop-report.html',
                            reportName: 'ZAP Security Reports',
                            reportTitles: 'Juice Shop Security Scan Results'
                        ])
                    } else {
                        error('ZAP HTML report was not generated.')
                    }
                }
            }
        }

        stage('DAST Security Gate') {
            steps {
                script {
                    if (!fileExists('dast-reports/zap-results.json')) {
                        error('ZAP JSON results not found!')
                    }

                    def zapResults = readJSON(
                        file: 'dast-reports/zap-results.json'
                    )

                    def highRiskCount = 0
                    def mediumRiskCount = 0

                    if (zapResults.site) {
                        zapResults.site.each { site ->
                            if (site.alerts) {
                                site.alerts.each { alert ->
                                    if (alert.riskdesc?.startsWith('High')) {
                                        highRiskCount++
                                    } else if (alert.riskdesc?.startsWith('Medium')) {
                                        mediumRiskCount++
                                    }
                                }
                            }
                        }
                    }

                    echo "========== ZAP RESULTS =========="
                    echo "High Risk: ${highRiskCount}"
                    echo "Medium Risk: ${mediumRiskCount}"
                    echo "================================="

                    if (highRiskCount > 0) {
                        unstable("High-risk vulnerabilities found: ${highRiskCount}")
                    } else if (mediumRiskCount > 5) {
                        unstable("Multiple medium-risk vulnerabilities found: ${mediumRiskCount}")
                    } else {
                        echo 'Security gate passed!'
                    }
                }
            }
        }

        stage('Pipeline Completed') {
            steps {
                echo 'DevSecOps pipeline execution completed!'
            }
        }
    }

    post {
        always {
            echo 'Cleaning up DAST Docker resources...'

            sh '''
                docker stop "$JUICE_SHOP" 2>/dev/null || true
                docker rm "$JUICE_SHOP" 2>/dev/null || true

                docker network rm "$DAST_NETWORK" 2>/dev/null || true
                docker volume rm "$REPORT_VOLUME" 2>/dev/null || true

                echo "DAST cleanup completed."
            '''
        }

        success {
            echo 'Pipeline execution: SUCCESS'
        }

        unstable {
            echo 'Pipeline execution completed with security warnings.'
        }

        failure {
            echo 'Pipeline execution: FAILURE'
        }
    }
}
