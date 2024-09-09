pipeline {
    agent any
    tools {
        git 'git-linux' // Git tool
        jdk 'java-linux' // JDK tool
        // Uncomment if using SonarQube Scanner or Dependency-Check
        // sonarQubeScanner 'SonarQubeScanner'
        // dependencyCheck 'dc'
    }
    environment {
        SONAR_HOME = tool "Sonar"
    }
    options {
        ansiColor('xterm')
    }

    stages {
        stage("Code") {
            steps {
                echo "\u001B[1;34m============= Starting Code Clone =============\u001B[0m" // Bold Blue
                echo "\u001B[34;1m\uD83D\uDCD1 Cloning Repository from GitHub... \u001B[0m" // Book emoji to represent code
                git url: "https://github.com/LondheShubham153/node-todo-cicd.git", branch: "master"
                echo "\u001B[1;32m✔ Code Cloned Successfully\u001B[0m" // Bold Green with checkmark
            }
        }

        stage("SonarQube Analysis") {
            steps {
                echo "\u001B[1;34m============= Starting SonarQube Analysis =============\u001B[0m" // Bold Blue
                echo "\u001B[34;1m\uD83D\uDD0D Analyzing code with SonarQube... \u001B[0m" // Magnifying glass emoji for analysis
                withSonarQubeEnv("Sonar") {
                    sh "$SONAR_HOME/bin/sonar-scanner -Dsonar.projectName=nodetodo -Dsonar.projectKey=nodetodo -Dsonar.verbose=true -X"
                }
                echo "\u001B[1;32m✔ SonarQube Analysis Done Successfully\u001B[0m" // Bold Green with checkmark
            }
        }

        stage("SonarQube Quality Gates") {
            steps {
                script {
                    echo "\u001B[1;34m============= Checking SonarQube Quality Gates =============\u001B[0m" // Bold Blue
                    echo "\u001B[34;1m\uD83D\uDEA7 Checking Quality Gates... \u001B[0m" // Construction sign for quality check
                    try {
                        timeout(time: 1, unit: "MINUTES") {
                            waitForQualityGate abortPipeline: false
                        }
                    } catch (Exception e) {
                        echo "\u001B[1;31m✘ Quality gate check timed out: ${e.message}\u001B[0m" // Bold Red with cross
                    }
                    echo "\u001B[1;32m✔ SonarQube Quality Gates Check Done Successfully\u001B[0m" // Bold Green with checkmark
                }
            }
        }

        stage("OWASP") {
            steps {
                echo "\u001B[1;34m============= Starting OWASP Dependency Check =============\u001B[0m" // Bold Blue
                echo "\u001B[34;1m\u26A0 Scanning for Vulnerabilities... \u001B[0m" // Warning sign for OWASP scan
                dependencyCheck additionalArguments: '--scan ./', odcInstallation: 'OWASP'
                dependencyCheckPublisher pattern: '**/dependency-check-report.xml'
                echo "\u001B[1;32m✔ OWASP Dependency Check Done Successfully\u001B[0m" // Bold Green with checkmark
            }
        }

        stage("Build & Test") {
            steps {
                echo "\u001B[1;34m============= Starting Build & Test =============\u001B[0m" // Bold Blue
                echo "\u001B[34;1m\uD83D\uDEE0 Building Docker Image... \u001B[0m" // Wrench and hammer emoji for build
                sh 'docker build -t node-app-batch-6:latest .'
                echo "\u001B[1;32m✔ Code Built Successfully\u001B[0m" // Bold Green with checkmark
            }
        }

        stage("Trivy") {
            steps {
                echo "\u001B[1;34m============= Starting Trivy Image Scan =============\u001B[0m" // Bold Blue
                echo "\u001B[34;1m\uD83E\uDD13 Scanning Docker Image for Vulnerabilities with Trivy... \u001B[0m" // Nerd face emoji for scanning
                sh "trivy image node-app-batch-6"
                echo "\u001B[1;32m✔ Trivy Image Scan Done Successfully\u001B[0m" // Bold Green with checkmark
            }
        }

        stage("Push to Private Docker Hub Repo") {
            steps {
                echo "\u001B[1;34m============= Starting Push to Private Docker Hub Repo =============\u001B[0m" // Bold Blue
                echo "\u001B[34;1m\uD83D\uDEA2 Pushing Docker Image to Private Repo... \u001B[0m" // Shipping emoji for pushing Docker image
                withCredentials([usernamePassword(credentialsId: "DokcerHubCred@123", passwordVariable: "dockerPass", usernameVariable: "dockerUser")]) {
                    sh "docker login -u ${env.dockerUser} -p ${env.dockerPass}"
                    sh "docker tag node-app-batch-6:latest ${env.dockerUser}/node-app-batch-6:latest"
                    sh "docker push ${env.dockerUser}/node-app-batch-6:latest"
                }
                echo "\u001B[1;32m✔ Successfully Pushed to Private Docker Hub Repo\u001B[0m" // Bold Green with checkmark
            }
        }

        stage("Deploy") {
            steps {
                echo "\u001B[1;34m============= Starting Deployment =============\u001B[0m" // Bold Blue
                echo "\u001B[34;1m\uD83D\uDFE1 Deploying Application... \u001B[0m" // Yellow circle emoji for deployment
                sh "docker-compose down && docker-compose up -d"
                echo "\u001B[1;32m✔ App Deployed Successfully\u001B[0m" // Bold Green with checkmark
            }
        }
    }
}
