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
                dockerTag = "RC-${env.BUILD_ID}"        
                customImage = docker.build("${imageName}:${dockerTag}")
                
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
            stage ('Push to Repo') {
            steps {
                dir('ArgoCD') {
                    withCredentials([gitUsernamePassword(credentialsId: 'git', gitToolName: 'Default')]) {
                        git branch: 'main', url: 'https://github.com/Panda-Academy-Core-2-0/ArgoCD.git'
                        sh """ cd frontend
                        sed -i "s#$imageName.*#$imageName:$dockerTag#g" frontend.yml
                        git commit -am "Set new $dockerTag tag."
                        git push origin main
                        """
                    }
                }
            }
        }
    }
       }
       
 post {
        always {
            junit testResults: "test-results/*.xml"
            cleanWs()
        }
//        success {
//            build job: 'app_of_apps', parameters: [ string(name: 'frontendDockerTag', value: "$dockerTag")], wait: false
//        }
    }
