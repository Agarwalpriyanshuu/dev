pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'MySonarQubeServer'
        PATH = "/opt/sonar-scanner/bin:$PATH"
        GIT_REPO_URL = 'https://github.com/Agarwalpriyanshuu/dev.git'
        GIT_CREDENTIALS_ID = 'git-token'
        SOURCE_BRANCH = 'dev'
        TARGET_BRANCH = 'master'
    }

    stages {
        stage('Checkout Source Branch') {
            steps {
                git branch: "${SOURCE_BRANCH}", url: "${GIT_REPO_URL}", credentialsId: "${GIT_CREDENTIALS_ID}"
            }
        }

        stage('Run Pytest') {
            steps {
                sh '''
                    python3 -m venv venv
                    . venv/bin/activate
                    curl -sS https://bootstrap.pypa.io/get-pip.py | python3
                    pip install -r requirements.txt
                    pytest --maxfail=1 --disable-warnings --junitxml=report.xml
                '''
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv("${SONARQUBE_ENV}") {
                    withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONAR_TOKEN')]) {
                        sh 'sonar-scanner -Dsonar.login=$SONAR_TOKEN'
                    }
                }
            }
        }

        stage('Merge to Master') {
            steps {
                script {
                    withCredentials([string(credentialsId: 'git-token', variable: 'GIT_PASSWORD')]) {
                        sh '''
                        set -e
                        echo "🔁 Configuring Git"
                        git config user.name "Jenkins"
                        git config user.email "jenkins@example.com"
        
                        echo "📥 Fetching branches"
                        git fetch origin
        
                        echo "🔀 Checking out master"
                        git checkout master || git checkout -b master origin/master
        
                        echo "📌 Pulling latest master"
                        git pull origin master
        
                        echo "🔁 Merging dev into master"
                        git merge origin/dev -m "Auto-merged dev into master after successful tests" || {
                            echo "❌ Merge conflict or error"
                            exit 1
                        }
        
                        echo "🚀 Pushing to GitHub"
                        git push https://Agarwalpriyanshuu:${GIT_PASSWORD}@github.com/Agarwalpriyanshuu/dev.git master
                        '''
                    }
                }
            }
        }


    post {
        success {
            echo "✅ All stages passed. Merged ${SOURCE_BRANCH} into ${TARGET_BRANCH}."
        }
        failure {
            echo "❌ Build failed. Merge aborted."
        }
        always {
            junit 'report.xml' // Publishes pytest results if using test reports
        }
    }
}
