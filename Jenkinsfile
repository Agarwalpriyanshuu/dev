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
                            git config user.name "Jenkins"
                            git config user.email "jenkins@example.com"

                            git fetch origin
                            git checkout ${TARGET_BRANCH} || git checkout -b ${TARGET_BRANCH} origin/${TARGET_BRANCH}
                            git pull origin ${TARGET_BRANCH}
                            git merge ${SOURCE_BRANCH} -m "✅ Auto-merged ${SOURCE_BRANCH} into ${TARGET_BRANCH} after tests and SonarQube analysis"
                            git push https://Agarwalpriyanshuu:${GIT_PASSWORD}@github.com/Agarwalpriyanshuu/dev.git ${TARGET_BRANCH}
                        '''
                    }
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
