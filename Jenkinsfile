pipeline {
  agent any

  options {
    timestamps()
    ansiColor('xterm')
  }

  environment {
    VENV_DIR = "${WORKSPACE}/.venv"
    PID_FILE = "${WORKSPACE}/app.pid"
    LOG_FILE = "${WORKSPACE}/app.log"
    APP_PORT = "8000"
  }

  stages {

    stage('Checkout') {
      steps {
        checkout scm
      }
    }

    stage('Setup venv + Install deps') {
      steps {
        sh '''
          set -euxo pipefail

          python3 --version

          # Create venv if not exists
          if [ ! -d "${VENV_DIR}" ]; then
            python3 -m venv "${VENV_DIR}"
          fi

          # Activate venv and install deps
          . "${VENV_DIR}/bin/activate"
          python -m pip install --upgrade pip
          pip install -r requirements.txt
        '''
      }
    }

    stage('Test') {
      steps {
        sh '''
          set -euxo pipefail
          . "${VENV_DIR}/bin/activate"

          # Run pytest only if tests exist (keeps it simple and non-breaking)
          if [ -d "tests" ] || ls -1 *test*.py >/dev/null 2>&1; then
            pytest -q
          else
            echo "No tests found, skipping pytest."
          fi
        '''
      }
    }

    stage('Run (Restart app)') {
      steps {
        sh '''
          set -euxo pipefail
          . "${VENV_DIR}/bin/activate"

          # Stop previous app instance if PID exists
          if [ -f "${PID_FILE}" ]; then
            OLD_PID=$(cat "${PID_FILE}" || true)
            if [ -n "${OLD_PID}" ] && kill -0 "${OLD_PID}" >/dev/null 2>&1; then
              echo "Stopping old app process: ${OLD_PID}"
              kill "${OLD_PID}" || true
              sleep 2
            fi
            rm -f "${PID_FILE}"
          fi

          # Start app in background
          nohup python3 app.py > "${LOG_FILE}" 2>&1 &
          echo $! > "${PID_FILE}"
          echo "Started app with PID $(cat ${PID_FILE})"

          # Quick health check (optional, but useful)
          sleep 2
          if command -v curl >/dev/null 2>&1; then
            curl -sSf "http://127.0.0.1:${APP_PORT}/" >/dev/null
            echo "✅ Health check passed on port ${APP_PORT}"
          else
            echo "curl not found; skipping health check."
          fi
        '''
      }
    }
  }

  post {
    always {
      archiveArtifacts artifacts: 'app.log, app.pid', allowEmptyArchive: true
      echo "Done. Log and PID archived (if present)."
    }
  }
}
