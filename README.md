# 🚀 Ansible + Azure CI/CD Pipeline
### Automated Java App Deployment Across Azure VMs Using Ansible Orchestration

![Azure](https://img.shields.io/badge/Azure-Cloud_Infrastructure-0078D4?style=flat&logo=microsoftazure&logoColor=white)
![Ansible](https://img.shields.io/badge/Ansible-Automation-EE0000?style=flat&logo=ansible&logoColor=white)
![Java](https://img.shields.io/badge/Java-PetClinic_App-ED8B00?style=flat&logo=openjdk&logoColor=white)
![Maven](https://img.shields.io/badge/Maven-Build_Tool-C71A36?style=flat&logo=apachemaven&logoColor=white)
![Tomcat](https://img.shields.io/badge/Tomcat-Web_Server-F8DC75?style=flat&logo=apachetomcat&logoColor=black)
![GitHub](https://img.shields.io/badge/GitHub-Source_Control-181717?style=flat&logo=github&logoColor=white)

---

## 📌 Project Overview

This project demonstrates a **production-style CI/CD pipeline** that integrates Azure, GitHub, and Ansible to automate full-cycle Java application deployment — from provisioning to a live running web server.

The pipeline eliminates manual deployment steps by orchestrating infrastructure provisioning, dependency installation, source code fetching, build, and deployment through a single Ansible control node.

---

## 🏗️ Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Microsoft Azure                          │
│                                                             │
│   ┌──────────────────┐       ┌─────────────────────────┐   │
│   │  Control Node    │──────▶│   Managed Node 1        │   │
│   │  (Ansible Host)  │       │   (Web Server - Tomcat) │   │
│   │                  │──────▶│   Managed Node 2        │   │
│   │  - SSH Keys      │       │   (Web Server - Tomcat) │   │
│   │  - Playbooks     │       └─────────────────────────┘   │
│   │  - Inventory     │                                      │
│   └──────────────────┘                                      │
│           │                                                  │
└───────────┼──────────────────────────────────────────────────┘
            │
            ▼
   ┌─────────────────┐
   │  GitHub Repo    │
   │  (PetClinic App)│
   └─────────────────┘
```

---

## 🛠️ Tech Stack

| Component | Tool | Purpose |
|---|---|---|
| Cloud Platform | Microsoft Azure | VM provisioning & networking |
| Automation | Ansible | Orchestration & deployment |
| Source Control | GitHub | Application source code |
| SSH Client | MobaXterm | Secure VM access & key management |
| Application | Spring PetClinic | Java web app |
| Build Tool | Apache Maven | Compile & package `.war` file |
| Web Server | Apache Tomcat | Application runtime |
| OS | Linux (Ubuntu) | All VMs |

---

## 📋 Prerequisites

- Azure subscription with VM creation permissions
- Ansible installed on the control node
- SSH key pair configured between control node and managed nodes
- Java JDK and Maven available on managed nodes
- Apache Tomcat installed on managed nodes

---

## 🖥️ Infrastructure Setup

### Azure VMs Provisioned

| VM Name | Role | Location | Size |
|---|---|---|---|
| `Controller-Node` | Ansible Control Node | Central India | Standard_B2ats |
| `Unmanaged-Node1` | Web Server | Central India | Standard_B2ats |
| `Managed-Node2` | Web Server | North Central US | Standard_B2ats |

### VM Configuration
All VMs run **Linux (Ubuntu)** with public IP addresses. SSH key-based authentication is configured between the control node and all managed nodes.

---

## ⚙️ Ansible Playbook — Deployment Steps

The deployment playbook automates the following sequence on both managed nodes:

```
1. Install Apache Tomcat
        ↓
2. Clone PetClinic source from GitHub
        ↓
3. Build .war file with Maven
        ↓
4. Deploy .war to Tomcat webapps directory
        ↓
5. Start / Restart Tomcat service
        ↓
6. ✅ App live in browser
```

### Sample Inventory (`hosts`)

```ini
[webservers]
<Managed-Node1-IP> ansible_user=azureuser ansible_ssh_private_key_file=~/.ssh/id_rsa
<Managed-Node2-IP> ansible_user=azureuser ansible_ssh_private_key_file=~/.ssh/id_rsa
```

### Sample Playbook (`deploy.yml`)

```yaml
---
- name: Deploy PetClinic to Web Servers
  hosts: webservers
  become: yes

  tasks:
    - name: Install Java JDK
      apt:
        name: default-jdk
        state: present
        update_cache: yes

    - name: Install Tomcat
      apt:
        name: tomcat9
        state: present

    - name: Clone PetClinic from GitHub
      git:
        repo: https://github.com/spring-projects/spring-petclinic.git
        dest: /opt/petclinic
        force: yes

    - name: Build the application with Maven
      command: mvn clean package -DskipTests
      args:
        chdir: /opt/petclinic

    - name: Deploy WAR to Tomcat
      copy:
        src: /opt/petclinic/target/spring-petclinic-*.war
        dest: /var/lib/tomcat9/webapps/petclinic.war
        remote_src: yes

    - name: Restart Tomcat
      service:
        name: tomcat9
        state: restarted
```

### Run the Playbook

```bash
ansible-playbook -i hosts deploy.yml
```

---

## 🔑 SSH Key Setup (MobaXterm)

1. Generate SSH key pair on the control node:
   ```bash
   ssh-keygen -t rsa -b 4096 -C "ansible-control"
   ```

2. Copy public key to each managed node:
   ```bash
   ssh-copy-id azureuser@<managed-node-ip>
   ```

3. Verify passwordless SSH:
   ```bash
   ansible all -i hosts -m ping
   ```

---

## 🌐 Access the Deployed Application

Once the playbook completes, the PetClinic app is accessible at:

```
http://<Managed-Node-Public-IP>:8080/petclinic
```

Both managed nodes serve the same application, demonstrating horizontal scalability.

---

## 📸 Screenshots

| Azure VM Dashboard | Ansible Terminal Output | Live App |
|---|---|---|
| ![Azure VMs](./screenshots/azure-vms.png) | ![Ansible run](./screenshots/ansible-run.png) | ![PetClinic](./screenshots/petclinic-live.png) |

> *Add your screenshots to a `/screenshots` folder in this repo.*

---

## 💡 Key Learnings

- **Idempotency**: Ansible ensures playbooks can be re-run safely without side effects
- **Agentless architecture**: No software needed on managed nodes beyond SSH + Python
- **Multi-node deployment**: A single playbook run deploys consistently across all target nodes
- **Version-controlled deployments**: Pulling source from GitHub ensures every deployment is traceable and reproducible

---

## 🔭 What's Next

- [ ] Add Ansible roles for better playbook organization
- [ ] Integrate GitHub Actions to trigger Ansible on every push
- [ ] Add health-check tasks post-deployment
- [ ] Set up Nginx as a reverse proxy / load balancer in front of the two nodes
- [ ] Use Azure Key Vault for secrets management

---

## 👤 Author

**[Your Name]**  
DevOps Engineer | Cloud & Automation Enthusiast  
[LinkedIn](https://linkedin.com/in/your-profile) • [GitHub](https://github.com/your-username)

---

> *"This project captures the essence of DevOps: automation, integration, and reliability."*
