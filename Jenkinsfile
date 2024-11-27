        def imageName="thatgreendragon/panda-frontend"
        def dockerRegistry=""
        def registryCredentials= "dockerhub"
        def dockerTag=""

pipeline {
    agent {
  label 'agent'
        }
        
    environment {
        PIP_BREAK_SYSTEM_PACKAGES=1
        scannerHome = tool 'SonarQube';
//        def imageName="thatgreendragon/panda-frontend"
//        def dockerRegistry=""
 //       def registryCredentials= "dockerhub"
        }

    stages {
        stage('Pobieranie z Gita') {
            steps {
                git branch: 'main', url: 'https://github.com/bpaszkowski/Frontend'
            }
        }
        stage('testy') {
            steps {
            sh '''pip3 install -r requirements.txt
                python3 -m pytest --cov=. --cov-report xml:test-results/coverage.xml --junitxml=test-results/pytest-report.xml
 
                '''
            }
        }
        stage('SonarQube analysis') {
            steps {
            withSonarQubeEnv('SonarQube') { // If you have configured more than one global server connection, you can specify its name
            sh "${scannerHome}/bin/sonar-scanner"
                }
            }
        }
        stage('buduje obraz dockerowy') {
            steps {
                script {
                dockerTag = "RC-${env.BUILD_ID}.${env.GIT_COMMIT.take(7)}"        
                customImage = docker.build("${imageName}:${env.BUILD_ID}")
                
                }
            }
            }
        stage('obraz do repo zdalnego'){
            steps {
                script {
                docker.withRegistry("${dockerRegistry}", "${registryCredentials}") {

            

                /* Push the container to the custom Registry */
                customImage.push()
                    }
                }
            }
             
         }
       }
       
       post {
            success {
    // One or more steps need to be included within each condition's block.
                    junit 'test-results/*.xml'
                    cleanWs()
                }
            }
    
    }
