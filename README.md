# Table of Contents
* [About](#about)
* [Requirements](#requirements)
* [Prepare execution for Docker Compose](#prepare-execution-for-docker-compose)
* [How to run Docker Compose](#how-to-run-docker-compose)
* [Prepare execution for Ansible playbooks](#prepare-execution-for-ansible-playbooks)
* [How to run Ansible playbooks](#how-to-run-ansible-playbooks)
* [Prepare Kubernetes deployment](#prepare-kubernetes-deployment)
* [How to deploy using Kubernetes](#how-to-deploy-using-kubernetes)
* [CI/CD](#cicd)
* [Related repositories](#related-repositories)

# About
Deployment repository of the myELGA agricultural compensation management platform. The repository contains the configuration required for:
- **VM configuration** using Ansible
- **Docker Compose** deployment
- **Kubernetes deployment** using MicroK8s
- **Automated deployment** through Jenkins

# Requirements
In order to execute the deployment processes, you must install the following tools:
* [ ] Install **Ansible**
    * Required to execute deployment playbooks
* [ ] Install **Docker**
    * Latest stable version = recommended
* [ ] Install **MicroK8s**
    * Required for Kubernetes deployment
* [ ] Install **Git**
    * Required to clone the repository
* [ ] Configure **SSH** access
    * Required for Ansible communication with the target VMs

# Prepare execution for Docker Compose
Before running the application with Docker Compose, create a local environment file based on the provided `.env.example` file:
```
cp .env.example .env
```
Open the created `.env` file and replace the example values with the actual configuration values and credentials.
> [!WARNING]
> The `.env` file contains sensitive configuration data and should not be committed to the repository.

# How to run Docker Compose
Start all application components using:
```
docker compose up -d
```
After the containers are running, the application can be accessed at:
<table>
    <thead>
        <tr>   
            <th>Component</th>
            <th>URL</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>backend</td>
            <td>http://localhost:8080</td>
        </tr>
        <tr>
            <td>frontend</td>
            <td>http://localhost:9000</td>
        </tr>
    </tbody>
</table>

# Prepare execution for Ansible playbooks
Before executing the Ansible playbooks, configure the inventory, deployment variables and SSH credentials used to connect to the target VMs. You can copy `all_example.yaml` to configure the variables file. The following local files are required:
- `ansible/inventory/hosts.yaml`
- `ansible/inventory/group_vars/all.yaml`
- `ansible/.ssh/devops_hua` 

> [!WARNING]
> Inventory variables and SSH private key should not be committed to the repository.

# How to run Ansible playbooks
The Ansible playbooks are located inside `ansible/playbooks/`. Run a playbook from the ansible directory using:
```
cd ansible

ansible-playbook playbooks/<playbook>.yaml
```

### Deploy application to 3 seperate VMs
Configure the application components on the separate VMs using: 
```
ansible-playbook playbooks/site.yaml
```

### Deploy application using Docker Compose
The Docker Compose deployment runs the complete myELGA application on the deployment VM. It can be executed through Ansible by running:
```
ansible-playbook playbooks/docker.yaml
```

### Configure Jenkins server
Configure the Jenkins VM using the following command from the `ansible` directory:
```
ansible-playbook playbooks/jenkins.yaml
```

# Prepare Kubernetes deployment
Before deploying the application to Kubernetes, create the Kubernetes secrets file based on the provided `kubernetes/secrets/secret.example.yaml` file:
```
cp kubernetes/secrets/secret.example.yaml kubernetes/secrets/secret.yaml
```
Open the created `secret.yaml` file and replace the example values with the actual configuration values and credentials.
> [!WARNING]
> The `secret.yaml` file contains sensitive configuration data and should not be committed to the repository.

# How to deploy using Kubernetes
The Kubernetes manifests are located inside the `kubernetes/` directory. The application can be deployed either manually using MicroK8s commands or automatically using Ansible.

### Deploy manually
Run the following commands from the repository root directory:
```
microk8s kubectl apply -f kubernetes/namespace.yaml
microk8s kubectl apply -f kubernetes/secrets/
microk8s kubectl apply -f kubernetes/database/
microk8s kubectl apply -f kubernetes/backend/
microk8s kubectl apply -f kubernetes/frontend/
microk8s kubectl apply -f kubernetes/ingress/
```

### Deploy using Ansible
The Kubernetes deployment can also be automated through Ansible. From the `ansible` directory run:
```
ansible-playbook playbooks/kubernetes.yaml
```

# CI/CD
The `Jenkinsfile` automates the deployment processes of the myELGA application. The pipeline supports different deployment actions through Jenkins parameters:

1. `CONFIGURE_VMS` executes the Ansible `site.yaml` playbook to configure and deploy the application components on the 3 separate VMs
2. `DEPLOY_DOCKER` executes the Ansible `docker.yaml` playbook to deploy application using Docker Compose
3. `COMPONENT` specifies the Kubernetes component that should be restarted after a new image is published

# Related repositories
<table>
    <thead>
        <tr>
            <th>Repository name</th>
            <th>URL</th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <td>myELGA_backend</td>
            <td>https://github.com/tolisApostolidis/myELGA_backend</td>
        </tr>
        <tr>
            <td>myELGA_frontend</td>
            <td>https://github.com/tolisApostolidis/myELGA_frontend</td>
        </tr>
        <tr>
            <td>myELGA_database</td>
            <td>https://github.com/tolisApostolidis/myELGA_database</td>
        </tr>
</table>