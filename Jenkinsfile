
pipeline {
    agent any

    triggers {
        pollSCM('H/2 * * * *')
    }
    environment {
        DAST_NETWORK = "dast-pipeline-${BUILD_NUMBER}"
        REPORT_VOLUME = "dast-reports-${BUILD_NUMBER}"
        JUICE_SHOP = "juice-shop-${BUILD_NUMBER}"

        TRIVY_IMAGE = "aquasec/trivy:0.76.2"
        JUICE_SHOP_SOURCE = "/var/jenkins_home/juice-shop-source"
        JENKINS_WORKSPACE = "/var/jenkins_home/workspace/DevSecOps-capstone"
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

                    echo "Checking shared Juice Shop source..."
                    test -f "$JUICE_SHOP_SOURCE/package.json"
                '''
            }
        }

        // ==========================================
        // SAST - SONARQUBE
        // ==========================================

        stage('SAST Scanning') {
            steps {
                script {
                    withSonarQubeEnv('SonarQube-Server') {
                        withCredentials([
                            string(
                                credentialsId: 'sonar-token',
                                variable: 'SONAR_TOKEN'
                            )
                        ]) {
                            sh '''
                                docker run --rm \
                                    --network devsecops-network \
                                    -e SONAR_HOST_URL=http://sonarqube:9000 \
                                    -e SONAR_TOKEN \
                                    -v jenkins_home:/var/jenkins_home \
                                    sonarsource/sonar-scanner-cli:latest \
                                    -Dsonar.projectBaseDir=/var/jenkins_home/juice-shop-source \
                                    -Dproject.settings=/var/jenkins_home/workspace/DevSecOps-capstone/SAST/sonar-project.properties \
                                    -Dsonar.working.directory=/var/jenkins_home/workspace/DevSecOps-capstone/.scannerwork
                            '''
                        }
                    }
                }
            }
        }

        stage('SAST Security Gate') {
            steps {
                timeout(time: 10, unit: 'MINUTES') {
                    script {
                        def qg = waitForQualityGate()

                        echo "SonarQube Quality Gate: ${qg.status}"

                        if (qg.status != 'OK') {
                            unstable(
                                "SAST Quality Gate failed: ${qg.status}"
                            )
                        } else {
                            echo 'SAST Security Gate PASSED!'
                        }
                    }
                }
            }
        }

        // ==========================================
        // SCA - TRIVY
        // ==========================================

        stage('SCA Dependency Preparation') {
            steps {
                sh '''
                    set -e

                    echo "Preparing Juice Shop dependencies..."

                    test -f "$JUICE_SHOP_SOURCE/package.json"

                    if [ ! -f "$JUICE_SHOP_SOURCE/package-lock.json" ]; then

                        echo "package-lock.json not found."
                        echo "Generating dependency lockfile using Node.js..."

                        docker run --rm \
                            -v jenkins_home:/var/jenkins_home \
                            -w "$JUICE_SHOP_SOURCE" \
                            node:22 \
                            sh -c '
                                node --version &&
                                npm --version &&
                                npm install --package-lock-only \
                                    --package-lock=true \
                                    --ignore-scripts \
                                    --no-audit \
                                    --no-fund
                            '

                    else
                        echo "Existing package-lock.json found."
                        echo "Skipping lockfile generation."
                    fi

                    echo "Validating dependency files..."

                    test -f "$JUICE_SHOP_SOURCE/package.json"
                    test -f "$JUICE_SHOP_SOURCE/package-lock.json"

                    echo "Dependency preparation completed."
                '''
            }
        }

        stage('SCA Scanning') {
            steps {
                sh '''
                    set -e

                    echo "Starting Trivy Software Composition Analysis..."

                    mkdir -p trivy-reports

                    docker volume create trivy-cache

                    echo "Generating Trivy JSON report..."

                    docker run --rm \
                        -v jenkins_home:/var/jenkins_home \
                        -v trivy-cache:/root/.cache/ \
                        "$TRIVY_IMAGE" \
                        fs \
                        --scanners vuln \
                        --format json \
                        --output /var/jenkins_home/workspace/DevSecOps-capstone/trivy-reports/trivy-report.json \
                        /var/jenkins_home/juice-shop-source

                    echo "Generating Trivy TXT report..."

                    docker run --rm \
                        -v jenkins_home:/var/jenkins_home \
                        -v trivy-cache:/root/.cache/ \
                        "$TRIVY_IMAGE" \
                        fs \
                        --scanners vuln \
                        --format table \
                        --output /var/jenkins_home/workspace/DevSecOps-capstone/trivy-reports/trivy-report.txt \
                        /var/jenkins_home/juice-shop-source

                    echo "Trivy scanning completed."

                    ls -lh trivy-reports/
                '''
            }
        }

        stage('SCA Security Gate & Reporting') {
            steps {
                script {
                    if (!fileExists('trivy-reports/trivy-report.json')) {
                        error('Trivy JSON report not found!')
                    }

                    def trivyResults = readJSON(
                        file: 'trivy-reports/trivy-report.json'
                    )

                    def criticalCount = 0
                    def highCount = 0
                    def mediumCount = 0
                    def lowCount = 0

                    if (trivyResults.Results) {
                        trivyResults.Results.each { result ->
                            if (result.Vulnerabilities) {
                                result.Vulnerabilities.each { vulnerability ->
                                    switch (vulnerability.Severity?.toUpperCase()) {
                                        case 'CRITICAL':
                                            criticalCount++
                                            break
                                        case 'HIGH':
                                            highCount++
                                            break
                                        case 'MEDIUM':
                                            mediumCount++
                                            break
                                        case 'LOW':
                                            lowCount++
                                            break
                                    }
                                }
                            }
                        }
                    }

                    def maxCritical = 0
                    def maxHigh = 10
                    def maxMedium = 30

                    def gatePassed =
                        criticalCount <= maxCritical &&
                        highCount <= maxHigh &&
                        mediumCount <= maxMedium

                    def gateStatus = gatePassed ? 'PASSED' : 'FAILED'

                    def gateReport = """
SCA SECURITY GATE RESULT: ${gateStatus}

Application: OWASP Juice Shop

Critical: ${criticalCount}
High: ${highCount}
Medium: ${mediumCount}
Low: ${lowCount}

Maximum Critical: ${maxCritical}
Maximum High: ${maxHigh}
Maximum Medium: ${maxMedium}
"""

                    echo gateReport

                    writeFile(
                        file: 'trivy-reports/security-gate-result.txt',
                        text: gateReport
                    )

                    def trivyText = readFile(
                        file: 'trivy-reports/trivy-report.txt'
                    )

                    def htmlReport = """
<!DOCTYPE html>
<html>
<head>
    <meta charset="UTF-8">
    <title>Trivy SCA Security Report</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            margin: 30px;
        }
        h1, h2 {
            color: #333;
        }
        pre {
            background: #f4f4f4;
            padding: 15px;
            overflow-x: auto;
            white-space: pre-wrap;
        }
        .status {
            font-weight: bold;
            font-size: 20px;
        }
    </style>
</head>
<body>

<h1>Trivy SCA Security Report</h1>

<p>Application: OWASP Juice Shop</p>
<p>Build Number: ${BUILD_NUMBER}</p>
<p>Security Gate Result:
   <span class="status">${gateStatus}</span>
</p>

<h2>Vulnerability Summary</h2>

<pre>${gateReport}</pre>

<h2>Trivy Vulnerability Scan Results</h2>

<pre>${trivyText}</pre>

</body>
</html>
"""

                    writeFile(
                        file: 'trivy-reports/trivy-report.html',
                        text: htmlReport
                    )

                    archiveArtifacts(
                        artifacts: 'trivy-reports/**',
                        allowEmptyArchive: false
                    )

                    publishHTML([
                        allowMissing: false,
                        alwaysLinkToLastBuild: true,
                        keepAll: true,
                        reportDir: 'trivy-reports',
                        reportFiles: 'trivy-report.html',
                        reportName: 'Trivy SCA Security Report',
                        reportTitles: 'Juice Shop SCA Results'
                    ])

                    if (!gatePassed) {
                        unstable('SCA Security Gate failed: vulnerability thresholds exceeded.')
                    } else {
                        echo 'SCA Security Gate PASSED!'
                    }
                }
            }
        }

        // ==========================================
        // DAST - OWASP ZAP
        // ==========================================

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
        stage('Push Reports to GitHub') {
            steps {
                withCredentials([usernamePassword(
                    credentialsId: 'github-read-token',
                    usernameVariable: 'GIT_USER',
                    passwordVariable: 'GIT_PASS'
                )]) {
                    sh '''
                        set -e

                        git config user.name "Jenkins CI"
                        git config user.email "jenkins@local.ci"

                        mkdir -p SCA/reports
                        cp -f trivy-reports/trivy-report.html SCA/reports/
                        cp -f trivy-reports/trivy-report.json SCA/reports/
                        cp -f trivy-reports/trivy-report.txt SCA/reports/
                        cp -f trivy-reports/security-gate-result.txt SCA/reports/

                        mkdir -p DAST/reports
                        cp -f dast-reports/zap-juice-shop-report.html DAST/reports/
                        cp -f dast-reports/zap-results.json DAST/reports/

                        git add SCA/reports/ DAST/reports/

                        if ! git diff --staged --quiet; then
                            git commit -m "chore: update SCA and DAST reports [Build #${BUILD_NUMBER}] [skip ci]"
                            git push https://${GIT_USER}:${GIT_PASS}@://github.com HEAD:main
                            echo "Reports pushed to GitHub successfully!"
                        else
                            echo "No report changes detected. Skipping GitHub push."
                        fi
                    '''
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
