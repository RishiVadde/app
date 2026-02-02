pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh '''
                    # Create virtual environment
                    python3 -m venv venv
                    . venv/bin/activate

                    # Install dependencies
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate

                    # Run pytest if tests exist
                    if [ -d "tests" ]; then
                        pytest -q
                    else
                        echo "No tests found, skipping."
                    fi
                '''
            }
        }

        stage('Deploy') {
            steps {
                sh '''
                    . venv/bin/activate

                    # Stop previous process if running
                    if [ -f app.pid ]; then
                        kill $(cat app.pid) || true
                        rm -f app.pid
                    fi

                    # Start the app
                    nohup python3 app.py > app.log 2>&1 &
                    echo $! > app.pid

                    echo "App deployed and running!"
                '''
            }
        }
    }
}
