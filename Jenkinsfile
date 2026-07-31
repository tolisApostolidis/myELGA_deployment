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

        stage('Prepare Ansible Files') {
            steps {
                withCredentials([
                    file(credentialsId: 'ansible-hosts', variable: 'HOSTS_FILE'),
                    file(credentialsId: 'ansible-vars', variable: 'VARS_FILE'),
                    file(credentialsId: 'ansible-private-key', variable: 'SSH_KEY_FILE')
                ]) {
                    sh '''
                        mkdir -p ansible/.ssh

                        cp "${HOSTS_FILE}" ansible/inventory/hosts.yaml
                        cp "${VARS_FILE}" ansible/inventory/group_vars/all.yaml
                        cp "${SSH_KEY_FILE}" ansible/.ssh/devops_hua
                    '''
                }
            }
        }
        
        stage('Test 3 VMs Connection') {
            when {
                expression { params.CONFIGURE_VMS }
            }

            steps {
                sh '''
                    ansible -i ansible/inventory/hosts.yaml database-vm,backend-vm,frontend-vm -m ping -e "ansible_ssh_private_key_file=${WORKSPACE}/ansible/.ssh/devops_hua"
                '''
            }
        }

        stage('Configure 3 VMs') {
            when {
                expression { params.CONFIGURE_VMS }
            }

            steps {
                sh '''
                    ansible-playbook -i ansible/inventory/hosts.yaml ansible/playbooks/site.yaml -e "ansible_ssh_private_key_file=${WORKSPACE}/ansible/.ssh/devops_hua"
                '''
            }
        }

        stage('Test Deployment VM Connection') {
            when {
                expression { params.DEPLOY_DOCKER }
            }

            steps {
                sh '''
                    ansible -i ansible/inventory/hosts.yaml deployment-vm -m ping -e "ansible_ssh_private_key_file=${WORKSPACE}/ansible/.ssh/devops_hua"
                ''' 
            }
        }

        stage('Deploy Docker Compose') {
            when {
                expression { params.DEPLOY_DOCKER }
            }

            steps {
                sh '''
                    ansible-playbook -i ansible/inventory/hosts.yaml ansible/playbooks/docker.yaml -e "ansible_ssh_private_key_file=${WORKSPACE}/ansible/.ssh/devops_hua"
                '''
            }
        }
    }

    post {

        success {
            echo 'Deployment pipeline completed successfully.'
        }

        failure {
            echo 'Deployment pipeline failed.'
        }

        always {
            sh '''
                rm -f ansible/hosts.yaml
                rm -f ansible/inventory/group_vars/all.yaml
                rm -f ansible/.ssh/devops_hua
            '''
        }
    }
}