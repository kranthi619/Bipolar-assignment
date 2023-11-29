pipeline {
    agent any

    tools {
        maven 'M2_HOME'
    }
  
    stages {
        stage('CheckOut') {
            steps {
                echo 'Checkout the source code from GitHub'
                git 'https://github.com/kranthi619/Bipolar-assignment.git'
            }
        }
        
        stage('Package the Application') {
            steps {
                echo "Packaging the Application"
                sh 'mvn clean package'
            }
        }
        
        stage('Publish Reports using HTML') {
            steps {
                publishHTML([allowMissing: false, alwaysLinkToLastBuild: false, keepAll: false, reportDir: '/var/lib/jenkins/workspace/Bipolar-assignment/target/surefire-reports', reportFiles: 'index.html', reportName: 'HTML Report', reportTitles: '', useWrapperFileDirectly: true]) 
            }
        }

        stage('Docker Image Creation') {
            steps {
                sh 'docker build -t kranthi619/insurence-project:latest .'
            }
        } 

        stage('Docker Login') {
            steps {
                withCredentials([usernamePassword(credentialsId: 'Docker-Hub', passwordVariable: 'dockerHubPassword', usernameVariable: 'dockerHubUser')]) {         
                    sh "docker login -u ${env.dockerHubUser} -p ${env.dockerHubPassword}"
                }
            }
        }

        stage('Push Image to DockerHub') {
            steps {
                sh 'docker push kranthi619/insurence-project:latest'
            }
        }

        stage('Deploy Application Ansible') {
            steps { 
                ansiblePlaybook credentialsId: 'privatekey', disableHostKeyChecking: true, installation: 'ansible', inventory: '/etc/ansible/hosts', playbook: 'deploy.yml'
            }
        }

        stage('Setup Monitoring') {
            steps {
                script {
                    echo 'Setting up basic monitoring...'

                    // Install Prometheus
                    sh 'docker run -d -p 9090:9090 --name prometheus prom/prometheus'

                    // Install Grafana
                    sh 'docker run -d -p 3000:3000 --name grafana grafana/grafana'

                    // Additional configuration or setup steps if needed

                    // Output monitoring URLs
                    echo 'Prometheus is running at http://localhost:9090'
                    echo 'Grafana is running at http://localhost:3000'

                    // Add any other monitoring tools or configurations here
                }
            }
        }
    } 
}

