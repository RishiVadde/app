pipeline {
    agent any

    stages {
        stage('Install') {
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
                    pytest -q
                '''
            }
        }

        stage('Run') {
            steps {
                sh '''
                    . venv/bin/activate
                    nohup python3 app.py > app.log 2>&1 &
                    echo "App started!"
                '''
            }
        }
    }
}
