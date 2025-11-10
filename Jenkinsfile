pipeline {
    agent any

    environment {
        // Use single quotes to avoid Groovy escape issues.
        BUILD_OUTPUT = 'C:\\Users\\swethasuresh\\works\\code'
    }

    stages {
        stage('Checkout') {
            steps {
                // Use checkout scm so Jenkins uses the repo/branch the job is configured with.
                checkout scm
            }
        }

        stage('Prepare Python env') {
            steps {
                bat '''
if not exist "C:\\jenkins_tools\\venvs\\project_venv\\Scripts\\python.exe" (
  py -3 -m venv C:\\jenkins_tools\\venvs\\project_venv
)
call "C:\\jenkins_tools\\venvs\\project_venv\\Scripts\\activate"
python -m pip install --upgrade pip
if exist "%WORKSPACE%\\requirements.txt" (
  pip install -r "%WORKSPACE%\\requirements.txt" --cache-dir "C:\\jenkins_tools\\.pip_cache"
) else (
  echo "requirements.txt not found, skipping pip install"
)
'''
            }
        }

        stage('Debug - list workspace') {
            steps {
                bat '''
@echo off
echo ====== WORKSPACE LISTING ======
dir "%WORKSPACE%" /B
echo ====== DETAILED ======
dir "%WORKSPACE%"
'''
            }
        }

       stage('Gather Build Files (zip)') {
    steps {
        bat '''@echo off
echo ==== GATHER BUILD FILES INTO ZIP (PowerShell single-line) ====
powershell -NoProfile -Command "Set-Location -LiteralPath '%WORKSPACE%'; if(-not(Test-Path 'build_artifacts')){ New-Item -ItemType Directory -Name 'build_artifacts' | Out-Null }; $files = @(); if(Test-Path 'AI-Data.csv'){ $files += (Resolve-Path 'AI-Data.csv').Path }; if(Test-Path 'Project.py'){ $files += (Resolve-Path 'Project.py').Path }; if(Test-Path 'requirements.txt'){ $files += (Resolve-Path 'requirements.txt').Path }; if(Test-Path 'README.md'){ $files += (Resolve-Path 'README.md').Path }; if($files.Count -eq 0){ Write-Output 'No build files found to package.'; exit 0 }; Remove-Item -Force -ErrorAction SilentlyContinue 'build_artifacts\\build_files.zip'; Compress-Archive -Path $files -DestinationPath 'build_artifacts\\build_files.zip' -Force; if(Test-Path 'build_artifacts\\build_files.zip'){ Write-Output ('Created ' + (Resolve-Path 'build_artifacts\\build_files.zip')) } else { Write-Error 'Failed to create build_files.zip'; exit 1 }"
'''
    }
}



        stage('Copy build zip to target') {
            steps {
                // Groovy expands ${env.BUILD_OUTPUT} here into the batch script.
                bat """
@echo off
echo ==== COPY BUILD ZIP TO TARGET ====
set "ARTIFACT_DIR=%WORKSPACE%\\build_artifacts"
set "TARGET_DIR=${env.BUILD_OUTPUT}"

if not exist "%ARTIFACT_DIR%" (
  echo Artifact dir missing: %ARTIFACT_DIR%
  exit /b 1
)

if not exist "%TARGET_DIR%" (
  echo Creating target dir: %TARGET_DIR%
  mkdir "%TARGET_DIR%"
)

REM Copy only the build zip
robocopy "%ARTIFACT_DIR%" "%TARGET_DIR%" build_files.zip /S /XO /R:2 /W:2 /NFL /NDL

set RC=%ERRORLEVEL%
echo Robocopy exit code: %RC%
if %RC% LEQ 7 (
  echo build_files.zip copied successfully to %TARGET_DIR%.
  exit /b 0
)
echo Copy failed with code %RC%.
exit /b %RC%
"""
            }
        }
    }

    post {
        success {
            echo 'Build completed and build_files.zip copied to target folder.'
        }
        failure {
            echo 'Build or copy failed!'
        }
    }
}
