# Ansible DevSecOps Playbooks 

This repository contains example Ansible playbooks designed to automate security tasks within a DevSecOps pipeline. These playbooks demonstrate how to integrate security checks and hardening procedures into your infrastructure automation process.

<img width="768" height="512" alt="image" src="https://github.com/user-attachments/assets/a5169df1-bb06-42e2-9623-9ce33df20d75" />

## What is an Ansible Playbook?

An Ansible Playbook is a YAML file that defines a set of automation tasks, configurations, and policies to be enforced on remote hosts. It allows you to describe your desired state for infrastructure and applications in a code-like manner, making it versionable, repeatable, and idempotent (meaning you can run it multiple times without causing unintended changes).

## Playbooks Overview

### 1. Host Hardening Playbook (`host-hardening.yml`)

This playbook performs basic security hardening on a Linux server by:
- Disabling SSH root login.
- Restricting SSH access to specific users.
- Disabling weak SSH encryption algorithms.
- Installing and configuring `fail2ban` to protect against brute-force attacks.
- Configuring a firewall (UFW) to allow only essential ports (SSH, HTTP, HTTPS).

### 2. Container & Dependency Scan Playbook (`container-dependency-scan.yml`)

This playbook is designed to run in a CI/CD pipeline and performs security scans on:
- **Docker Images**: Uses `Trivy` to scan container images for known vulnerabilities (CVEs), failing the pipeline if critical issues are found.
- **Python Dependencies**: Uses `Safety` to check for vulnerabilities in Python packages listed in a `requirements.txt` file.

## Prerequisites

- Ansible installed on the control node.
- For `host-hardening.yml`: Target hosts must be accessible via SSH.
- For `container-dependency-scan.yml`: Docker and Python/pip must be installed on the runner.

## Usage

### 1. Clone the Repository
```bash
git clone <your-repo-url>
cd ansible-devsecops-playbooks
```

### 2. Create an Inventory File
Create a file named `inventory.ini` to define your target hosts:
```ini
[webservers]
web-server-1 ansible_host=192.168.1.10
web-server-2 ansible_host=192.168.1.11

[local]
localhost ansible_connection=local
```

### 3. Run the Host Hardening Playbook
```bash
ansible-playbook -i inventory.ini host-hardening.yml --user <your-username> --become
```

### 4. Run the Security Scan Playbook (in CI/CD)
Example command to run the scan locally:
```bash
ansible-playbook -i inventory.ini container-dependency-scan.yml -e "image_name=my-app:latest requirements_path=./requirements.txt"
```

## Example Integration in GitLab CI

```yaml
stages:
  - security

trivy_scan:
  stage: security
  image: docker:latest
  services:
    - docker:dind
  before_script:
    - apk add --no-cache ansible
  script:
    - ansible-playbook -i localhost, -c local container-dependency-scan.yml
  artifacts:
    paths:
      - ./*security-scan-report*.txt
```

## Key Benefits

- **Infrastructure as Code (IaC)**: Security policies are defined in code, versioned, and reviewable.
- **Automation**: Eliminates manual security checks, reducing human error.
- **Shift-Left Security**: Identifies vulnerabilities early in the development lifecycle.
- **Compliance**: Helps enforce consistent security configurations across all environments.

## Contributing

Feel free to submit issues, fork the repository, and create pull requests to improve these playbooks.

## License

This project is licensed under the MIT License.

---

