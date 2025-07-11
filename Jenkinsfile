pipeline {
    agent { label 'slave-01' }

    tools {
        maven "maven"
    }

    stages {
        stage('SCM Checkout') {
            steps {
                git 'https://github.com/Rohitpotdar01/BankingApp.git'
            }
        }

        stage('Maven Build') {
            steps {
                sh "mvn -Dmaven.test.failure.ignore=true clean package"
            }
        }

        stage('Docker Build & Push') {
            steps {
                sh 'docker version'
                sh "docker build -t rohitpotdar/bankapp-eta-app:${BUILD_NUMBER} ."
                sh "docker tag rohitpotdar/bankapp-eta-app:${BUILD_NUMBER} rohitpotdar/bankapp-eta-app:latest"
                
                withCredentials([usernamePassword(credentialsId: 'dock-password', usernameVariable: 'DOCKERHUB_USR', passwordVariable: 'DOCKERHUB_PSW')]) {
                    sh "echo $DOCKERHUB_PSW | docker login -u $DOCKERHUB_USR --password-stdin"
                }

                sh "docker push rohitpotdar/bankapp-eta-app:latest"
            }
        }

        stage('Deploy to Test') {
            steps {
                sh "ansible-playbook -i /etc/ansible/hosts ${WORKSPACE}/ansible/playbooks/deploy_test.yaml"
            }
        }

        stage('Verify Test Deployment') {
            steps {
                sh "ansible-playbook -i /etc/ansible/hosts ${WORKSPACE}/ansible/playbooks/verify_test.yaml"
            }
        }

        stage('Deploy to Prod') {
            when {
                expression {
                    currentBuild.currentResult == 'SUCCESS'
                }
            }
            steps {
                sh "ansible-playbook -i /etc/ansible/hosts ${WORKSPACE}/ansible/playbooks/deploy_prod.yaml"
            }
        }
    }
}
