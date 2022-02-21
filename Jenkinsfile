pipeline {
    agent any

    // Install the Jenkins tools you need for your project / environment
    tools {
        maven 'maven' // Refers to a global tool configuration for Maven called 'maven-3.8.3'
    }

    // Pull your Snyk token from a Jenkins encrypted credential
    // (type "Secret text"... see https://jenkins.io/doc/book/using/using-credentials/#adding-new-global-credentials)
    // and put it in temporary environment variable for the Snyk CLI to consume.
    environment {
        SNYK_TOKEN = credentials('SNYK_TOKEN')
    }

    stages {

        stage('Initialize & Cleanup Workspace') {
            steps {
               echo 'Initialize & Cleanup Workspace'
               sh 'ls -la'
               sh 'rm -rf *'
               sh 'rm -rf .git'
               sh 'rm -rf .gitignore'
               sh 'ls -la'
            }
        }

        stage('Git Clone') {
            steps {
                //git url: 'https://github.com/lmaeda/java-goofs.git'
                git url: 'https://github.com/lmaeda/BingAds-Java-SDK.git'

                sh 'ls -la'
            }
        }

        stage('Test Build Requirements') {
            steps {
                sh 'java -version'
                sh 'mvn -v'
            }
        }

        // Not required if just install the Snyk CLI on your Agent
        stage('Download Snyk CLI') {
            steps {
                sh '''
                    latest_version=$(curl -Is "https://github.com/snyk/snyk/releases/latest" | grep "^location" | sed s#.*tag/##g | tr -d "\r")
                    echo "Latest Snyk CLI Version: ${latest_version}"
                    snyk_cli_dl_linux="https://github.com/snyk/snyk/releases/download/${latest_version}/snyk-linux"
                    echo "Download URL: ${snyk_cli_dl_linux}"
                    curl -Lo ./snyk "${snyk_cli_dl_linux}"
                    chmod +x snyk
                    ls -la
                    ./snyk -v
                '''
            }
        }

        stage('Build') {
            steps {
              sh 'mvn -e -X package'
              //sh './mvnw test -Dsnyk.skip'
            }
        }

        // Run snyk test to check for vulnerabilities and fail the build if any are found
        // Consider using --severity-threshold=<low|medium|high> for more granularity (see snyk help for more info).
        stage('Snyk Test using Snyk CLI') {
            steps {
                //sh './snyk test --fail-on=upgradable --severity-threshold=high --project-name=java-goofs --org=demo_high --target-reference=docker-tomcat'
                sh './snyk test --fail-on=upgradable --severity-threshold=critical --project-name=BingAds-Java-SDK --org=demo_high --target-reference=main'
            }
        }

        // Capture the dependency tree for ongoing monitoring in Snyk.
        // This is typically done after deployment to some environment (ex staging, test, production, etc).
        stage('Snyk Monitor using Snyk CLI') {
            steps {
                // Use your own Snyk Organization with --org=<your-org>
                //sh './snyk monitor --fail-on=upgradable --severity-threshold=critical --project-name=java-goofs --org=demo_high --target-reference=docker-tomcat'
                sh './snyk monitor --fail-on=upgradable --severity-threshold=critical --project-name=BingAds-Java-SDK --org=demo_high --target-reference=main'
            }
        }

        // Capture the dependency tree for ongoing monitoring in Snyk.
        // This is typically done after deployment to some environment (ex staging, test, production, etc).
        stage('Snyk Container Test using Snyk CLI') {
            steps {
                // Use your own Snyk Organization with --org=<your-org>
                //sh './snyk container test --severity-threshold=critical --project-name=java-goofs javagoof:orig'
                sh './snyk container test --severity-threshold=critical --project-name=snyk-demo-blueocean docker:dind'
            }
        }
        
        // Capture the dependency tree for ongoing monitoring in Snyk.
        // This is typically done after deployment to some environment (ex staging, test, production, etc).
        stage('Snyk Container Monitor using Snyk CLI') {
            steps {
                // Use your own Snyk Organization with --org=<your-org>
                //sh './snyk container monitor --severity-threshold=critical --project-name=java-goofs javagoof:orig'
                sh './snyk container monitor --severity-threshold=critical --project-name=snyk-demo-blueocean docker:dind'
            }
        }
        
    }
}
