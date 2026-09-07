pipeline {
    agent any

    options {
        timestamps()
        buildDiscarder(logRotator(numToKeepStr: '10'))
    }

    stages {

        stage('Checkout Source Code') {
            steps {
                echo '=== STAGE 1: CHECKOUT - fetching the Selenium test project from GitHub ==='
                checkout scm
                bat 'dir'
            }
        }

        stage('Environment Check') {
            steps {
                echo '=== STAGE 2: ENVIRONMENT CHECK - locating Python and Chrome ==='
                bat '''
                    @echo off
                    ver
                    echo --- python on PATH ---
                    where python
                    python --version
                    echo --- py launcher ---
                    where py
                    echo --- chrome ---
                    if exist "C:\Program Files\Google\Chrome\Application\chrome.exe" echo CHROME FOUND
                    if exist "C:\Program Files (x86)\Google\Chrome\Application\chrome.exe" echo CHROME FOUND X86
                    exit /b 0
                '''
            }
        }

        stage('Set Up Environment & Dependencies') {
            steps {
                echo '=== STAGE 3: SETUP - creating the virtual environment and installing packages ==='
                bat '''
                    @echo off
                    echo [1/3] Creating virtual environment...
                    if exist venv rmdir /s /q venv
                    python -m venv venv
                    echo [2/3] Upgrading pip...
                    venv\Scripts\python.exe -m pip install --upgrade pip
                    echo [3/3] Installing testing packages...
                    venv\Scripts\python.exe -m pip install -r requirements.txt
                    venv\Scripts\python.exe -m pip list
                '''
            }
        }

        stage('Execute Selenium Tests') {
            steps {
                echo '=== STAGE 4: TEST - running the Selenium test suite with pytest ==='
                bat '''
                    @echo off
                    if not exist reports mkdir reports
                    echo Running Pytest Suite...
                    venv\Scripts\python.exe -m pytest tests/ -v --junitxml=reports/junit-report.xml
                '''
            }
        }
    }

    post {
        always {
            junit testResults: 'reports/junit-report.xml', allowEmptyResults: true
            archiveArtifacts artifacts: 'reports/*.xml', allowEmptyArchive: true
        }
        success { echo "SELENIUM PIPELINE SUCCESS - build #${env.BUILD_NUMBER}" }
        failure { echo 'SELENIUM PIPELINE FAILED - check the console output above.' }
    }
}
