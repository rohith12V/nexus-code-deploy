pipeline {
	agent any
	tools {
	    maven "MAVEN3"
	    jdk "OracleJDK17"
	}

	stages {
        // Stage - 1
	    stage('Fetch code') {
            steps {
                git branch: 'main', url: 'https://github.com/rohith12V/nexus-code-deploy.git'
            }
        }

        // Stage - 2
        stage('Build') {
            steps {
                sh 'mvn clean install -DskipTests'
            }

            post {
               success {
                  echo 'Now Archiving it...'
                  archiveArtifacts artifacts: '**/target/*.jar'
               }
            }
        }

        // Stage - 3
        stage('UNIT TEST') {
         steps {
              sh 'mvn test'
            }
        }

        // Stage - 4
        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }

        // Stage - 5
        stage('Sonar Analysis') {
            environment {
                scannerHome = tool 'sonarqube'
            }
            steps {
                withSonarQubeEnv('sonarqube') {
                    sh '''${scannerHome}/bin/sonar-scanner -Dsonar.projectKey=vprofile \
                    -Dsonar.projectName=vprofile \
                    -Dsonar.projectVersion=1.0 \
                    -Dsonar.sources=src/ \
                    -Dsonar.java.binaries=target/classes/com/bezkoder/spring/restapi/ \
                    -Dsonar.junit.reportsPath=target/surefire-reports/ \
                    -Dsonar.jacoco.reportsPath=target/jacoco.exec \
                    -Dsonar.java.checkstyle.reportPaths=target/checkstyle-result.xml'''
                }
            }
        }

        // # Stage - 6
        stage("Quality Gate") {
                steps {
                    timeout(time: 1, unit: 'HOURS') {
                        // true = set pipeline to UNSTABLE, false = don't
                        waitForQualityGate abortPipeline: true
                    }
                }
            }

        // Define a stage named "UploadArtifact"
        stage("UploadArtifact") {
            // Within the stage, define the steps to execute
            steps {
                // Use the nexusArtifactUploader step to upload an artifact to Nexus
                nexusArtifactUploader(
                    // Specify the version of Nexus being used
                    nexusVersion: 'nexus3',
                    // Set the protocol for communication with Nexus (HTTP in this case)
                    protocol: 'http',
                    // Provide the URL of the Nexus server
                    nexusUrl: '172.31.46.23:8081',
                    // Group ID for the artifact in Nexus
                    groupId: 'QA',
                    // Dynamically generate version using build ID and timestamp
                    version: "${env.BUILD_ID}-${env.BUILD_TIMESTAMP}",
                    // Name of the repository in Nexus to upload to
                    repository: 'vprofile-repo',
                    // Credentials ID for accessing Nexus (configured elsewhere in Jenkins)
                    credentialsId: 'nexuslogin',
                    // Define the artifact to upload
                    artifacts: [
                        [
                            // Artifact ID for the artifact
                            artifactId: 'vproapp',
                            // Classifier for the artifact (optional)
                            classifier: '',
                            // Path to the artifact file
                            file: 'target/spring-boot-3-rest-api-example-0.0.1-SNAPSHOT.jar',
                            // Type of the artifact (war in this case)
                            type: 'jar'
                        ]
                    ]
                )
            }
        }



    }
}