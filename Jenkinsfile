pipeline {
    agent any

    stages {

        stage('Build') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    pip install --upgrade pip
                    pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    . venv/bin/activate
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

                    if [ -f app.pid ]; then
                        kill $(cat app.pid) || true
                        rm -f app.pid
                    fi

                    nohup python3 app.py > app.log 2>&1 &
                    echo $! > app.pid

                    echo "App deployed and running!"
                '''
            }
        }
    }

    post {
        always {
            // This makes artifacts appear in Jenkins
            archiveArtifacts artifacts: 'app.log, app.pid', allowEmptyArchive: true
        }
    }
}
