pipeline {

    /* Jenkins can run this pipeline on any available worker */
    agent any

    /* Variables used everywhere in pipeline */
    environment {
        APP_DIR = "/srv/poetry_django"
        RELEASE_DIR = "${APP_DIR}/releases/${BUILD_ID}"
        POETRY_BIN = "/var/lib/jenkins/.local/bin"
    }

    stages {

        stage('Checkout Code') {
            steps {
                echo "Pulling code from Git repository"
                checkout scm
            }
        }

        stage('Create Release Folder') {
            steps {
                sh '''
                mkdir -p $RELEASE_DIR
                '''
            }
        }

        stage('Copy Code to Release') {
            steps {
                sh '''
                rsync -av --exclude=.git ./ $RELEASE_DIR/
                '''
            }
        }

        stage('Setup Poetry') {
            steps {
                sh '''
                export PATH="$POETRY_BIN:$PATH"
                poetry --version
                '''
            }
        }

        stage('Install Dependencies') {
            steps {
                sh '''
                export PATH="$POETRY_BIN:$PATH"
                cd $RELEASE_DIR
                poetry config virtualenvs.in-project true
                poetry install --no-interaction
                '''
            }
        }

        stage('Django Migrations & Static') {
            steps {
                sh '''
                export PATH="$POETRY_BIN:$PATH"
                cd $RELEASE_DIR
                poetry run python manage.py migrate
                poetry run python manage.py collectstatic --noinput
                '''
            }
        }

        stage('Switch Symlink') {
            steps {
                sh '''
                ln -sfn $RELEASE_DIR $APP_DIR/current
                '''
            }
        }

        stage('Restart Gunicorn') {
            steps {
                sh '''
                sudo systemctl restart poetry_django.service
                '''
            }
        }
    }

    post {
        success {
            echo "✅ Deployment completed successfully"
        }
        failure {
            echo "❌ Deployment failed"
        }
    }
}
