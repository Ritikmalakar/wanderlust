pipeline {
    agent any

    environment {
        SONAR_HOME = tool "Sonar"
    }

    stages {

        stage("Clone Code from Github") {
            steps {
                git url: "https://github.com/Ritikmalakar/wanderlust.git",
                    branch: "main"
            }
        }

        stage("SonarQube Quality Analysis") {
            steps {
                withSonarQubeEnv("Sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=wanderlust -Dsonar.projectKey=wanderlust"
                }
            }
        }

        stage("OWASP Dependency Check") {
            steps {
                withCredentials([
                    string(
                        credentialsId: 'NVD API Key',
                        variable: 'NVD_API_KEY'
                    )
                ]) {
                    dependencyCheck(
                        additionalArguments: "--scan ./ --nvdApiKey ${NVD_API_KEY}",
                        odcInstallation: 'dc'
                    )
                }

                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
            }
        }

        stage("Sonar Quality Gate Scan") {
            steps {
                timeout(time: 2, unit: "MINUTES") {
                    waitForQualityGate abortPipeline: false
                }
            }
        }

        stage("Trivy File System Scan") {
            steps {
                sh "trivy fs --format table -o trivy-fs-report.txt ."
            }
        }

        stage("Deploy using Docker Compose") {
            steps {
                sh "docker compose up -d"
            }
        }
    }
}
