pipeline {
    agent any
    stages {
        stage('Build Backend') {
            steps {
                sh 'mvn clean package -DskipTests=true'
            }
        }
        stage('Unit Tests') {
            steps {
                sh 'mvn test'
            }
        }
        stage('Sonar Analysis') {
            environment {
                scannerHome =  tool 'SONAR_SCANNER'
            }
            steps {
                withSonarQubeEnv('SONAR_LOCAL') {
                    sh "${scannerHome}/bin/sonar-scanner -e -Dsonar.projectKey=DeployBack -Dsonar.host.url=http://localhost:9000 -Dsonar.login=1fb7f696d68e7f21ae673e6748f4ab895336c23d -Dsonar.java.binaries=target -Dsonar.coverage.exclusions=**/.mvn/**,**/src/test/**,**/model/**,**Application.java"
                }
            }
        }
        stage('Quality Gate') {
            steps {
                sleep(10)
                timeout(time: 1, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }
        stage('Deploy Backend') {
            steps {
                deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks-backend', war: 'target/tasks-backend.war'
            }
        }
        stage('API Test') {
            steps {
                dir('api-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/limberger/tasks-api-test'                                
                    sh 'mvn test'
                }

            }
        }
        stage('Deploy Frontend') {
            steps {
                dir('frontend') {
                    git credentialsId: 'github_login', url: 'https://github.com/limberger/tasks-frontend'                                
                    sh 'mvn test'
                    deploy adapters: [tomcat8(credentialsId: 'TomcatLogin', path: '', url: 'http://localhost:8001/')], contextPath: 'tasks', war: 'target/tasks.war'
                }
            }
        }

        stage('Functional Test') {
            steps {
                dir('functional-test') {
                    git credentialsId: 'github_login', url: 'https://github.com/limberger/tasks-functional-test'                                
                    sh 'mvn test'
                }

            }
        }
        stage('Deploy Prod') {
            steps{
                sh 'docker-compose build'
                sh 'docker-compose up -d'
            }
        }
        stage('Smoke Test') {
            steps {
                sleep(10)
                dir('functional-test') {
                    sh 'mvn verify -Dskip.surefire.test'
                }

            }
        }

    }
    post {
        always {
            junit allowEmptyResults: true, testResults: 'target/surefire-reports/*.xml,api-test/target/surefire-reports/*.xml,functional-test/target/surefire-reports/*.xml,functional-test/target/failsafe-reports/*.xml'

        }

    }
}



