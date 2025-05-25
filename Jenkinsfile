pipeline {
    agent any

    environment {
        SONARQUBE_ENV = 'MySonarQubeServer'
        PATH = "/opt/sonar-scanner/bin:$PATH"
    }

    stages {
        stage('Checkout Code') {
            steps {
                git branch: 'dev', url: 'https://github.com/Agarwalpriyanshuu/dev.git', credentialsId: 'git-token'
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
                        git checkout master || git checkout -b master origin/master
                        git pull origin master
                        git merge dev -m "Auto-merged dev into master after SonarQube analysis"
        
                        git push https://Agarwalpriyanshuu:${GIT_PASSWORD}@github.com/Agarwalpriyanshuu/dev.git master
                        '''
                    }
                }
            }
        }
    }
}
