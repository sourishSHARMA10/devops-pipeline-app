
pipeline {
    agent any

    stages {
        stage('Build') {
            steps {
                echo 'Installing Python dependencies...'
                sh '''
                    python3 -m venv venv
                    ./venv/bin/pip install --upgrade pip
                    ./venv/bin/pip install -r requirements.txt
                '''
            }
        }

        stage('Test') {
            steps {
                echo 'Running unit tests...'
                sh './venv/bin/python -m unittest discover -s .'
            }
        }

        stage('Deploy') {
            steps {
                echo 'Copying application to deployment directory...'
                sh '''
                    mkdir -p "$WORKSPACE/python-app-deploy"
                    cp app.py "$WORKSPACE/python-app-deploy/"
                '''
            }
        }

        stage('Run Application') {
            steps {
                echo 'Starting Flask application...'
                sh '''
                    if [ -f "$WORKSPACE/python-app-deploy/app.pid" ]; then
                        kill "$(cat "$WORKSPACE/python-app-deploy/app.pid")" 2>/dev/null || true
                    fi

                    BUILD_ID=dontKillMe nohup "$WORKSPACE/venv/bin/python" \
                        "$WORKSPACE/python-app-deploy/app.py" \
                        > "$WORKSPACE/python-app-deploy/app.log" 2>&1 &

                    echo $! > "$WORKSPACE/python-app-deploy/app.pid"
                '''
            }
        }

        stage('Test Application') {
            steps {
                echo 'Checking the running application...'
                sh '''
                    for i in $(seq 1 15); do
                        if curl -fsS http://127.0.0.1:5000/; then
                            echo
                            exit 0
                        fi
                        sleep 2
                    done

                    cat "$WORKSPACE/python-app-deploy/app.log"
                    exit 1
                '''
            }
        }
    }

    post {
        success {
            echo 'Pipeline completed successfully!'
        }
        failure {
            echo 'Pipeline failed. Check the logs for details.'
        }
    }
}
