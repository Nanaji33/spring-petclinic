pipeline {
    agent any
    stages{
        
        stage("buildmycode"){
            steps {
                sh """
                 ./mvnw package -DskipTests
                 ls -lrt target/*.jar
                """
            }
        }

        stage("run-unit-tests"){
            steps {
                sh """
                ./mvnw test
                """
            }
        }

        stage("sonarscan"){
            steps {
                
               script {
                    withSonarQubeEnv( installationName: 'dev-sonar-01', credentialsId: 'jenkins-sonar-creds') {
                        sh """
                            sonar-scanner \
                                        -Dsonar.projectKey=petclinic \
                                        -Dsonar.projectName=petclinic \
                                        -Dsonar.exclusions=**/*.java \
                                        -Dsonar.sources=. \
                                        -Dsonar.java.binaries=target/classes \
                                        -Dsonar.sourceEncoding=UTF-8
                        """
                    }
                }
            }
        }
    }
}