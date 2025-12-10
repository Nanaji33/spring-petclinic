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

        stage("Sonar Quality Gate") {
            steps {
                script {
                    timeout(time: 5, unit: 'MINUTES') {
                        def qg = waitForQualityGate()
                        if (qg.status != 'OK') {
                            error "Pipeline failed due to quality gate: ${qg.status}"
                        }
                    }
                }
            }
        }

         stage("Upload-Artifacts-Nexus"){
            steps {
                script {
                   def pom = new XmlSlurper().parse(new File("${env.WORKSPACE}/pom.xml"))
                    def groupId = pom.groupId.text() ?: pom.parent.groupId.text()
                    def artifactId = pom.artifactId.text()
                    def version = pom.version.text()
                    // Path to the Maven-built JAR
                    env.JAR_FILE_PATH = "${env.WORKSPACE}/target/${artifactId}-${version}.jar"
                    echo "GroupId: ${groupId}, ArtifactId: ${artifactId}, Version: ${version}"
                    echo "JAR Path: ${env.JAR_FILE_PATH}"

                    nexusArtifactUploader(
                        nexusVersion: 'nexus3',
                        protocol: 'http',
                        nexusUrl: 'nexus:8081',
                        groupId: 'groupId',
                        version: 'Version',
                        repository: 'maven-snapshots',
                        credentialsId: 'jenkins-nexus-creds',
                        artifacts: [
                            [
                                artifactId: 'artifactId',
                                file: 'env.JAR_FILE_PATH'
                                type: "jar",
                                classifier: ""
                            ]
                        ]
                    )

                }

            }
        }

         stage("Deploy-Dev"){
            steps {
                echo "Deploying....Dev Servers..."
            }
        }

        stage("Deploy-UAT"){
            steps {
                echo "Deploying....UAT Servers..."
            }
        }

        stage("Deploy-PROD"){
            steps {
                echo "Deploying....PROD Servers..."
            }
        }
    }
}