# Terraform और Ansible — Beginner to Advanced Guide

> DevOps, Cloud Engineer और Platform Engineer बनने के लिए सम्पूर्ण हिन्दी गाइड

---

# विषय सूची

1. Infrastructure as Code (IaC) क्या है?
2. Terraform क्या है?
3. Ansible क्या है?
4. Terraform vs Ansible
5. दोनों साथ में कैसे काम करते हैं?
6. Terraform Installation
7. Ansible Installation
8. Terraform Basics
9. Terraform State
10. Terraform Variables
11. Terraform Outputs
12. Terraform Modules
13. Terraform Workspaces
14. Remote State
15. Terraform Best Practices
16. Ansible Basics
17. Inventory
18. Playbooks
19. Variables
20. Roles
21. Templates
22. Handlers
23. Vault
24. Dynamic Inventory
25. Terraform + Ansible Integration
26. Real Production Example
27. CI/CD Integration
28. Best Practices
29. Interview Questions
30. Learning Roadmap

---

# अध्याय 1 : Infrastructure as Code (IaC)

## पहले क्या होता था?

मान लीजिए AWS में आपको

* 5 EC2
* 2 Database
* 1 Load Balancer
* VPC
* Subnets

बनाने हैं।

पहले इंजीनियर AWS Console खोलकर एक-एक चीज़ बनाते थे।

इसमें समय भी लगता था और गलती की संभावना भी रहती थी।

---

## Infrastructure as Code

अब हम Infrastructure को Code में लिखते हैं।

जैसे

```text
Server = Code

Database = Code

Network = Code
```

एक कमांड चलाइए

```bash
terraform apply
```

और पूरी Infrastructure अपने आप बन जाएगी।

---

# अध्याय 2 : Terraform क्या है?

Terraform HashiCorp का Infrastructure as Code Tool है।

इसका मुख्य काम Cloud Resources बनाना है।

उदाहरण

* AWS EC2
* Azure VM
* Google Cloud VM
* Kubernetes Cluster
* VPC
* Security Groups
* S3 Bucket
* Database

Terraform Infrastructure Provision करता है।

---

# अध्याय 3 : Ansible क्या है?

Terraform ने Server बना दिया।

अब उस Server पर

* Docker Install करना
* Java Install करना
* Node Install करना
* Nginx Install करना
* Application Deploy करना

ये सब Ansible करता है।

---

# अध्याय 4 : Terraform vs Ansible

| Terraform               | Ansible                  |
| ----------------------- | ------------------------ |
| Infrastructure बनाता है | Server Configure करता है |
| Cloud Resources         | Software Installation    |
| State रखता है           | Stateless                |
| HCL Language            | YAML                     |
| Provisioning            | Configuration            |

---

# अध्याय 5 : Workflow

```text
Developer
      │
      ▼
Terraform
      │
      ▼
Cloud Infrastructure
      │
      ▼
Ansible
      │
      ▼
Software Installation
      │
      ▼
Application Deployment
```

---

# अध्याय 6 : Terraform Installation

Windows

```bash
choco install terraform
```

Linux

```bash
sudo apt install terraform
```

Mac

```bash
brew install terraform
```

---

# अध्याय 7 : पहला Terraform Program

```hcl
resource "aws_instance" "web" {

  ami = "ami-xxxxx"

  instance_type = "t2.micro"

}
```

Commands

```bash
terraform init

terraform plan

terraform apply

terraform destroy
```

---

# अध्याय 8 : Terraform State

Terraform हर Resource का रिकॉर्ड रखता है।

```
terraform.tfstate
```

इसमें लिखा होता है

* कौन सा Server बना
* कौन सा Database बना
* किसका IP क्या है

---

# अध्याय 9 : Variables

```hcl
variable "instance_type" {

 default="t2.micro"

}
```

---

# अध्याय 10 : Outputs

```hcl
output "public_ip" {

 value=aws_instance.web.public_ip

}
```

---

# अध्याय 11 : Modules

बार-बार Code लिखने के बजाय

Reusable Module बनाइए।

उदाहरण

```
EC2 Module

VPC Module

RDS Module
```

---

# अध्याय 12 : Workspaces

Development

Testing

Production

सभी के लिए अलग-अलग State।

---

# अध्याय 13 : Remote State

Production में State File Laptop पर नहीं रखते।

बल्कि

AWS S3

Azure Storage

Terraform Cloud

में रखते हैं।

---

# अध्याय 14 : Terraform Best Practices

* छोटे Modules बनाइए
* Variables का उपयोग करें
* Secrets Git में Commit न करें
* Remote Backend रखें
* Version Lock करें

---

# अध्याय 15 : Ansible Installation

Ubuntu

```bash
sudo apt install ansible
```

Check

```bash
ansible --version
```

---

# अध्याय 16 : Inventory

```
[web]

192.168.1.20

192.168.1.21

192.168.1.22
```

---

# अध्याय 17 : Playbook

```yaml
- hosts: web

  become: yes

  tasks:

    - name: Install nginx

      apt:

        name: nginx

        state: present
```

---

# अध्याय 18 : Variables

```yaml
vars:

 app_name: ride-booking
```

---

# अध्याय 19 : Roles

Role Structure

```
roles/

 web/

 database/

 redis/

 docker/
```

---

# अध्याय 20 : Templates

Dynamic Configuration

```
nginx.conf.j2
```

---

# अध्याय 21 : Handlers

जब Configuration बदले

तभी Service Restart हो।

---

# अध्याय 22 : Vault

Passwords Encrypt करने के लिए।

```
ansible-vault
```

---

# अध्याय 23 : Dynamic Inventory

AWS

Azure

GCP

से Servers अपने आप Detect हो जाते हैं।

---

# अध्याय 24 : Terraform + Ansible

Terraform

↓

Infrastructure

↓

Inventory Generate

↓

Ansible

↓

Deployment

---

# अध्याय 25 : Production Example

Ride Booking App

Terraform

* VPC
* Private Subnet
* Public Subnet
* EC2
* RDS
* Redis
* Load Balancer

Ansible

* Docker
* Node
* Nginx
* SSL
* PM2
* Backend
* Frontend

---

# अध्याय 26 : CI/CD

Git Push

↓

GitHub Actions

↓

Terraform Plan

↓

Terraform Apply

↓

Ansible Playbook

↓

Deployment Complete

---

# अध्याय 27 : Best Practices

* Git का उपयोग करें
* State Lock रखें
* Secrets Encrypt करें
* छोटे Modules बनाएं
* Environment अलग रखें
* Reusable Roles बनाएं

---

# अध्याय 28 : Interview Questions

### Terraform क्या है?

Infrastructure as Code Tool

---

### Terraform State क्या है?

Infrastructure का रिकॉर्ड।

---

### Terraform Module क्या है?

Reusable Infrastructure।

---

### Terraform Backend क्या है?

जहाँ State Store होती है।

---

### Terraform Workspace क्या है?

Multiple Environments Manage करने के लिए।

---

### Ansible Inventory क्या है?

Servers की List।

---

### Playbook क्या है?

Automation Script।

---

### Role क्या है?

Reusable Configuration।

---

### Handler क्या है?

Service Restart जैसी Triggered Action।

---

### Vault क्या है?

Secrets Encryption Tool।

---

# अध्याय 29 : Learning Roadmap

```
Linux
        │
        ▼
Networking
        │
        ▼
Git
        │
        ▼
Docker
        │
        ▼
Terraform
        │
        ▼
Ansible
        │
        ▼
AWS
        │
        ▼
Kubernetes
        │
        ▼
Helm
        │
        ▼
GitHub Actions
        │
        ▼
Production DevOps
```

---

# निष्कर्ष

**Terraform** Infrastructure बनाता है।

**Ansible** उस Infrastructure को Configure करता है।

दोनों मिलकर आधुनिक DevOps Automation की मजबूत नींव तैयार करते हैं।
