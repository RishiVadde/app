pipeline {
    agent any

    stages {
        stage('Install') {
            steps {
                sh '''
                    python3 -m pip install --user -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                sh '''
                    python3 -m pytest -q
                '''
            }
        }

        stage('Run App') {
            steps {
                sh '''
                    nohup python3 app.py > app.log 2>&1 &
                    echo "App started on port 8000"
                '''
            }
        }
    }
}
