#  DevOps Pipeline Lab — Containerized Web Deployment

A hands-on DevOps project demonstrating a complete CI/CD pipeline using **Git, Docker, Terraform, Ansible, and Jenkins** to deploy a containerized web application on AWS.

---

##  Architecture Overview

```
Developer (Local Machine)
        │
        ├── Git Push → GitHub Repository
        │                      │
        │                      ▼
        │              Jenkins Pipeline
        │              ┌──────────────┐
        │              │ 1. Checkout  │
        │              │ 2. Build     │
        │              │ 3. Test      │
        │              │ 4. Push      │
        │              └──────┬───────┘
        │                     │
        ▼                     ▼
   Terraform            Docker Hub
  (AWS EC2 VM)         (Image Registry)
        │                     │
        └──────── Ansible ────┘
                  (Pull & Run Container)
                        │
                        ▼
                Live Website on AWS
              http://3.108.66.66
```

---

##  Tech Stack

| Tool | Purpose |
|------|---------|
| **Git & GitHub** | Version control and source of truth |
| **Docker** | Containerize the web application with Nginx |
| **Docker Hub** | Container image registry |
| **Terraform** | Provision AWS EC2 infrastructure as code |
| **Ansible** | Configure VM and deploy Docker container |
| **Jenkins** | Automate the full CI/CD pipeline |
| **AWS EC2** | Cloud virtual machine (t3.micro, ap-south-1) |
| **Nginx** | Web server inside the container |

---

##  Project Structure

```
devops-lab/
├── index.html              # Static web page served by Nginx
├── Dockerfile              # Container definition
├── Jenkinsfile             # CI/CD pipeline definition
├── ansible/
│   ├── inventory.ini       # EC2 host configuration
│   └── deploy.yml          # Ansible playbook
└── app/
    └── terraform/
        ├── main.tf         # EC2 + Security Group resources
        ├── variables.tf    # Input variable definitions
        └── terraform.tfvars # Variable values
```

---

##  Part 1 — Git & Docker

### What it does
Packages a static HTML page inside an Nginx container.

### Dockerfile
```dockerfile
FROM nginx:alpine
RUN rm -rf /usr/share/nginx/html/*
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
```

### Run locally
```bash
# Build the image
docker build -t mizhabnp/devops-lab:latest .

# Run on port 8080
docker run -d -p 8080:80 --name devops-lab mizhabnp/devops-lab:latest

# Visit http://localhost:8080
```

### Push to Docker Hub
```bash
docker push mizhabnp/devops-lab:latest
```

---

##  Part 2 — Terraform & AWS

### What it does
Provisions an AWS EC2 instance with Docker pre-installed via a user data script.

### Resources created
- **Security Group** — allows inbound SSH (22) and HTTP (80)
- **EC2 Instance** — t3.micro, Ubuntu 22.04, Mumbai region (ap-south-1)

### Commands
```bash
cd app/terraform

terraform init      # Initialize providers
terraform plan      # Preview changes
terraform apply     # Create infrastructure
```

### Output
```
Outputs:
public_ip = "3.108.66.66"
```

---

##  Part 3 — Ansible

### What it does
SSHs into the EC2 instance, pulls the Docker image from Docker Hub, and runs the container.

### Inventory
```ini
[webservers]
3.108.66.66 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/devops-key.pem
```

### Run the playbook
```bash
cd ansible

# Test connectivity
ansible -i inventory.ini webservers -m ping

# Deploy the container
ansible-playbook -i inventory.ini deploy.yml
```

### Playbook tasks
1. Ensure Docker service is running
2. Pull `mizhabnp/devops-lab:latest` from Docker Hub
3. Remove any existing container
4. Run container on port 80 with restart policy

---

##  Part 4 — Jenkins CI/CD Pipeline

### What it does
Automates the entire build and push process on every GitHub commit.

### Pipeline stages
```
Checkout → Build Docker Image → Test → Push to Docker Hub
```

### Jenkinsfile stages
| Stage | Action |
|-------|--------|
| Checkout | Jenkins pulls latest code from GitHub |
| Build | `docker build -t mizhabnp/devops-lab:latest .` |
| Test | Verifies build success with echo |
| Push | Pushes image to Docker Hub using stored credentials |

### Jenkins setup
```bash
# Run Jenkins locally via Docker
docker run -d -p 8090:8080 \
  -v jenkins_home:/var/jenkins_home \
  -v //var/run/docker.sock:/var/run/docker.sock \
  --name jenkins jenkins/jenkins:lts
```

Then open `http://localhost:8090` and configure a Pipeline job pointing to this repository.

---

##  Success Criteria

- [x] Repository with clean commit history
- [x] Docker image builds and runs locally on port 8080
- [x] Image pushed to Docker Hub
- [x] `terraform apply` completes without errors
- [x] EC2 instance running in AWS Console
- [x] Website accessible at `http://3.108.66.66`
- [x] Ansible playbook deploys container to EC2
- [x] Jenkins pipeline shows **SUCCESS** status
- [x] Code change triggers automatic pipeline run

---

##  Quick Start

### Prerequisites
- Git, Docker Desktop
- AWS CLI configured (`aws configure`)
- Terraform installed
- Ansible (via WSL on Windows)
- Jenkins running locally

### Deploy from scratch
```bash
# 1. Clone the repo
git clone https://github.com/EmperorWhiteBeard/devops-lab
cd devops-lab

# 2. Build and test locally
docker build -t mizhabnp/devops-lab:latest .
docker run -d -p 8080:80 mizhabnp/devops-lab:latest

# 3. Provision cloud infrastructure
cd app/terraform
terraform init && terraform apply

# 4. Deploy to cloud via Ansible (in WSL)
cd ../../ansible
ansible-playbook -i inventory.ini deploy.yml

# 5. Visit your live site
echo "Site is live at http://$(terraform -chdir=../app/terraform output -raw public_ip)"
```

---

##  Author

**Mizhab Mujeeb NP**
- GitHub: [@EmperorWhiteBeard](https://github.com/EmperorWhiteBeard)
- LinkedIn: [mizhabnp](https://www.linkedin.com/in/mizhabnp)
- Email: mizhabnp@gmail.com

---

*Built as part of a DevOps hands-on lab covering containerization, infrastructure as code, configuration management, and CI/CD automation.*
