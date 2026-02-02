pipeline {
    agent any

    options {
        timestamps()
        disableConcurrentBuilds()
    }

    environment {
        VENV_DIR = "${WORKSPACE}/venv"
        LOG_FILE = "${WORKSPACE}/app.log"
        PID_FILE = "${WORKSPACE}/app.pid"
        APP_PORT = "8000"
    }

    stages {

        stage('Build') {
            steps {
                sh '''
                    set -e

                    echo "Python version:"
                    python3 --version

                    echo "Creating virtual environment..."
                    rm -rf "${VENV_DIR}"
                    python3 -m venv "${VENV_DIR}"

                    echo "Installing dependencies..."
                    . "${VENV_DIR}/bin/activate"
                    python -m pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    set -e
                    . "${VENV_DIR}/bin/activate"

                    if [ -d "tests" ]; then
                        echo "Running tests..."
                        pytest -q
                    else
                        echo "No tests folder found. Skipping tests."
                    fi
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    set -e
                    . "${VENV_DIR}/bin/activate"

                    echo "Stopping old app if running..."
                    if [ -f "${PID_FILE}" ]; then
                        OLD_PID=$(cat "${PID_FILE}" || true)
                        if [ -n "${OLD_PID}" ] && kill -0 "${OLD_PID}" >/dev/null 2>&1; then
                            kill "${OLD_PID}" || true
                            sleep 2
                        fi
                        rm -f "${PID_FILE}"
                    fi

                    echo "Starting app..."
                    nohup python3 app.py > "${LOG_FILE}" 2>&1 &
                    echo $! > "${PID_FILE}"

                    echo "App started with PID: $(cat ${PID_FILE})"
                    echo "Waiting 2 seconds..."
                    sleep 2

                    echo "Health check (local):"
                    if command -v curl >/dev/null 2>&1; then
                        curl -sSf "http://127.0.0.1:${APP_PORT}/" >/dev/null
                        echo "Health check passed on port ${APP_PORT}"
                    else
                        echo "curl not installed, skipping health check."
                    fi

                    echo "Last lines of app.log:"
                    tail -n 20 "${LOG_FILE}" || true
                '''
            }
        }
    }

    post {
        always {
            archiveArtifacts artifacts: 'app.log, app.pid', allowEmptyArchive: true
        }

        success {
            echo "CI/CD completed successfully."
        }

        failure {
            echo "CI/CD failed. Check console log and archived app.log."
        }
    }
}
