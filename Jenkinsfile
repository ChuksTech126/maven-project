pipeline {
    agent any

    tools {
        maven 'Maven-3.9.12'
    }

    environment {
        SONAR_SCANNER = tool 'sonarqube-scanner'
    }

    stages {

        stage('Compile Code') {
            steps {
                sh 'mvn clean compile'
            }
        }

        stage('PMD Code Review') {
            steps {
                sh 'mvn -P metrics pmd:pmd'
            }
            post {
                success {
                    recordIssues tools: [pmdParser(pattern: '**/pmd.xml')]
                }
            }
        }

        stage('SonarQube Analysis') {
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh "${SONAR_SCANNER}/bin/sonar-scanner"
                }
            }
        }

        stage('Quality Gate') {
            steps {
                timeout(time: 3, unit: 'MINUTES') {
                    waitForQualityGate abortPipeline: true
                }
            }
        }

        stage('Package App') {
            steps {
                sh 'mvn package'
            }
        }

        stage('Publish Artifact to JFrog') {
            steps {
                rtUpload(
                    serverId: 'jfrog-dev',
                    spec: '''{
                        "files": [
                            {
                                "pattern": "target/kitchensink.war",
                                "target": "new/"
                            }
                        ]
                    }'''
                )
            }
        }

        stage('Deploy and Run Docker on EC2') {
            steps {
                ansiblePlaybook(
                    credentialsId: 'ec2-ssh-key',
                    disableHostKeyChecking: true,
                    installation: 'ansible',
                    inventory: 'inventory',
                    playbook: 'playbook.yml',
                    extras: '--ssh-extra-args="-o StrictHostKeyChecking=no"'
                )
            }
        }
    }
}
