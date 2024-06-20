# AnsiblewithJenkins

# Docker Container Automation Project

## Project Overview
The objective of this project is to automate the process of building, pushing, and deploying Docker containers using GitHub, Jenkins, Ansible, and Docker.

## Tools and Technologies
1. **GitHub:** Source code repository
2. **Jenkins:** Continuous integration server
3. **Ansible:** Automation tool
4. **Docker:** Container platform
5. **Docker Hub:** Container registry

## Prerequisites
- GitHub account and repository
- AWS account with necessary permissions
- Jenkins server setup with necessary plugins
- Ansible installed on Jenkins server
- Docker installed on both Jenkins server and Docker host
- Docker Hub account

## Step-by-Step Process

### 1. Developer Pushes Dockerfile to GitHub
- Create a GitHub repository.
- Add a Dockerfile to the repository.
- [My GitHub repository link](https://github.com/praveenkpraveen31/simple-docker-project.git)

### 2. Create 3 EC2 Instances on AWS
- Enable port 8080 on your security group or allow all TCP traffic on your security group.

### 3. Setting Up Jenkins Server

#### On Jenkins Host:
1. **Set Hostname**
    ```bash
    hostnamectl set-hostname jenkins-host
    ```
2. **Set Root Password**
    ```bash
    passwd root
    ```
3. **Install Java**
    ```bash
    yum install java* -y
    ```
4. **Add Jenkins Repository**
    ```bash
    sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
    ```
5. **Import Jenkins Repository Key**
    ```bash
    sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io-2023.key
    ```
6. **Update System Packages**
    ```bash
    sudo yum upgrade
    ```
7. **Install Jenkins**
    ```bash
    yum install jenkins -y
    ```
8. **Enable and Start Jenkins Service**
    ```bash
    systemctl enable --now jenkins.service
    ```
9. **Check Jenkins Service Status**
    ```bash
    systemctl status jenkins.service
    ```
10. **Install Git**
    ```bash
    yum install git -y
    ```
11. **Generate SSH Key and Transfer to Ansible Server**
    - Generate SSH key and enable SSH password authentication for root user.

#### Jenkins Setup on the Browser
1. **Access Jenkins:**
    - Open a web browser and navigate to `http://your_server_ip_or_domain:8080`.
    - You will see the Jenkins unlock screen.
2. **Unlock Jenkins:**
    ```bash
    cat /var/lib/jenkins/secrets/initialAdminPassword
    ```
    - Copy the password and paste it into the unlock screen.
3. **Install Suggested Plugins:**
    - Choose "Install suggested plugins" to automatically install the most common plugins.
4. **Create Admin User:**
    - Set up your first admin user and complete the configuration.
5. **Jenkins Dashboard:**
    - After the setup is complete, you will be redirected to the Jenkins dashboard.
6. **Install SSH Plugin:**
    - Go to Jenkins Dashboard → Plugins → Available Plugins and search for "Public Over SSH" plugin and install.

### 4. Configure Jenkins to Use SSH
- Add Jenkins and Ansible server SSH authentication in Jenkins server.

#### Jenkins Job Configuration:
- **Source Code Management: Git**
    ```text
    Repository URL: https://github.com/praveenkpraveen31/simple-docker-project.git
    ```
- **Build Steps:**
    - Add "Send files or execute commands over SSH".
    - Add Jenkins server and write the following rsync command in the "Exec Command" section:
    ```bash
    rsync -avz /path/to/local/file user@remote_host:/path/to/remote/directory
    ```
    - Add Ansible server and write the following commands in the "Exec Command" section:
    ```bash
    cd /opt
    docker image build -t $JOB_NAME:$BUILD_ID .
    docker image tag $JOB_NAME:$BUILD_ID praveeen267/$JOB_NAME:$BUILD_ID
    docker image tag $JOB_NAME:$BUILD_ID praveeen267/$JOB_NAME:latest
    docker image push praveeen267/$JOB_NAME:$BUILD_ID
    docker image push praveeen267/$JOB_NAME:latest
    docker image rmi $JOB_NAME:$BUILD_ID
    docker image rmi praveeen267/$JOB_NAME:$BUILD_ID
    docker image rmi praveeen267/$JOB_NAME:latest
    docker image rmi rockylinux:8
    ```

### 5. Setting Up Ansible Server

#### On Ansible Host:
1. **Set Root Password**
    ```bash
    sudo passwd root
    ```
2. **Enable Root Login with Password**
    ```bash
    sudo vim /etc/ssh/sshd_config
    PermitRootLogin yes
    PasswordAuthentication yes
    sudo systemctl restart sshd
    ```
3. **Verify Root Login**
    ```bash
    ssh root@your_ansible_host_ip
    ```

#### Install Ansible and Docker:
1. **Update Package List**
    ```bash
    sudo yum update -y
    ```
2. **Enable EPEL Repository**
    ```bash
    sudo amazon-linux-extras install epel -y
    ```
3. **Install Ansible**
    ```bash
    sudo yum install -y ansible
    ```
4. **Verify Ansible Installation**
    ```bash
    ansible --version
    ```

#### Generate SSH Key and Transfer to Docker Host:
1. **Generate SSH Key Pair**
    ```bash
    ssh-keygen
    ```
2. **Transfer SSH Key**
    ```bash
    ssh-copy-id root@docker_host_ip
    ```

#### Configure Ansible Hosts:
1. **Edit Ansible Hosts File**
    ```bash
    sudo vim /etc/ansible/hosts
    Docker_host_ip
    ```

#### Create Ansible Playbook:
1. **Create Playbook Directory**
    ```bash
    mkdir /root/playbooks
    cd /root/playbooks
    vim docker.yaml
    ```
2. **Playbook Content**
    ```yaml
    ---
    - name: docker-project
      hosts: all
      tasks:
        - name: stopping container
          shell: docker container stop web100
          ignore_errors: yes
        - name: removing container
          shell: docker container rm web100
          ignore_errors: yes
        - name: removing image
          shell: docker image rmi praveeen267/docker_project
          ignore_errors: yes
        - name: creating new container
          shell: docker run -d --name web100 -p 8080:80 praveeen267/docker_project
          ignore_errors: yes
    ```

### 6. Setting Up Docker Host Server

#### On Docker Host:
1. **Set Root Password**
    ```bash
    sudo passwd root
    ```
2. **Enable Root Login with Password**
    ```bash
    sudo vim /etc/ssh/sshd_config
    PermitRootLogin yes
    PasswordAuthentication yes
    sudo systemctl restart sshd
    ```

#### Install Docker:
1. **Install Docker**
    ```bash
    amazon-linux-extras install docker -y
    ```
2. **Start Docker Service**
    ```bash
    systemctl start docker
    ```
3. **Enable Docker to Start on Boot**
    ```bash
    systemctl enable docker
    ```
4. **Verify Docker Installation**
    ```bash
    docker --version
    ```
5. **Run Test Container**
    ```bash
    docker run hello-world
    ```

### Final Step
- Log in to your Jenkins Dashboard, navigate to your project, and click **Build Now**.

---

## Author
Diksha Shirke

---

For any queries or issues, please contact [DikshaShirke](mailto:dikshasshirke@gmail.com).
