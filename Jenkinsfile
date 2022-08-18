pipeline {
    agent any
    stages {
        stage ('Build Backend') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage ('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }  
        stage ('Sonar Analysis') {
            environment {
                scannerHome = tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://IP:9000 -Dsonar.login=TOKEN -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**application.java"
                }                
            }
        }
        stage ('Deploy Backend') {
            steps {
                deploy adapters: [tomcat9(credentialsId: 'LoginTomcat', path: '', url: 'http://IP/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        } 
        stage ('API Test') {
            steps {
                dir('api-test') {
                    git credentialsId: 'LoginGitHub', url: 'https://github.com/julioserra/tasks-api-test'
                    sh 'mvn test'
                }
            }
        }
        stage ('Deploy Frontend') {
            steps {
                dir('frontend') {
                    git credentialsId: 'LoginGitHub', url: 'https://github.com/julioserra/tasks-frontend'
                    sh 'mvn clean package'
                    deploy adapters: [tomcat9(credentialsId: 'LoginTomcat', path: '', url: 'http://IP/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }                             
    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml, api-test/target/surefire-reports/*.xml'
            archiveArtifacts artifacts: 'target/tasks-backend.war, frontend/target/tasks.war', onlyIfSuccessful: true
        }
        unsuccessful {
            emailext attachLog: true, body: 'See the attached log below', subject: 'Build Falhou', to: 'MEUEMAIL'
        }
        fixed {
            emailext attachLog: true, body: 'See the attached log below', subject: 'Build está completo', to: 'MEUEMAIL'
        }
    }
}