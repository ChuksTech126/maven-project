pipeline {
    agent any

    tools {
        maven 'Maven-3.9.12'
    }

    stages {

        stage('Compile Code') {
            steps {
                sh '/opt/maven/bin/mvn clean compile'
            }
        }

        stage('PMD Code Review') {
            steps {
                sh '/opt/maven/bin/mvn -P metrics pmd:pmd'
            }
            post {
                success {
                    recordIssues tools: [pmdParser(pattern: '**/pmd.xml')]
                }
            }
        }

        stage('SonarQube Analysis') {
            environment {
                scannerHome = tool 'sonarqube-scanner'
            }
            steps {
                withSonarQubeEnv('SonarQube-Server') {
                    sh "${scannerHome}/bin/sonar-scanner"
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
                sh '/opt/maven/bin/mvn package'
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

        stage('Deploy with Ansible') {
            steps {
                ansiblePlaybook(
                    credentialsId: 'ec2-id',
                    disableHostKeyChecking: true,
                    installation: 'ansible',
                    inventory: 'inventory',
                    playbook: 'playbook.yml'
                )
            }
        }

        stage('Build & Run Docker Container') {
            steps {
                sh '''
                docker ps -q --filter "name=myapp" | xargs -r docker stop
                docker ps -aq --filter "name=myapp" | xargs -r docker rm

                docker build -t bloomy/myapp:1.0.${BUILD_NUMBER} .
                docker run -d -p 8050:8050 --name myapp bloomy/myapp:1.0.${BUILD_NUMBER}
                '''
            }
        }
    }
}
