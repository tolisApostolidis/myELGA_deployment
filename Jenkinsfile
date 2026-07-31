pipeline {

    agent any

    parameters {
        booleanParam(
            name: 'CONFIGURE_VMS',
            defaultValue: false,
            description: 'Run site.yaml to configure the 3 VMs'
        )

        booleanParam(
            name: 'DEPLOY_DOCKER',
            defaultValue: true,
            description: 'Run docker.yaml to deploy the application with Docker Compose'
        )
    }

    environment {
        ANSIBLE_CONFIG = "${WORKSPACE}/ansible/ansible.cfg"
    }

    stages {
        
        stage('Test 3 VMs Connection') {
            when {
                expression { params.CONFIGURE_VMS }
            }

            steps {
                sh '''
                    ansible -i ansible/hosts.yaml database-vm,backend-vm,frontend-vm -m ping
                '''
            }
        }

        stage('Configure 3 VMs') {
            when {
                expression { params.CONFIGURE_VMS }
            }

            steps {
                sh '''
                    ansible-playbook -i ansible/hosts.yaml ansible/site.yaml
                '''
            }
        }

        stage('Test Deployment VM Connection') {
            when {
                expression { params.DEPLOY_DOCKER }
            }

            steps {
                sh '''
                    ansible -i ansible/hosts.yaml deployment-vm -m ping
                '''
            }
        }

        stage('Deploy Docker Compose') {
            when {
                expression { params.DEPLOY_DOCKER }
            }

            steps {
                sh '''
                    ansible-playbook -i ansible/hosts.yaml ansible/docker.yaml
                '''
            }
        }
    }
}