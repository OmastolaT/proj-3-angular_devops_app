# Demo: Angular Static Site + Jenkins + Ansible + MariaDB

## Overview
This repository layout is prepared for your in-demo pipeline:
- Ansible controller runs on **ANS** (16.16.201.56)
- Jenkins will be installed on **JNK** (13.60.9.2)
- Angular static site will be deployed to **UBT1** (13.53.37.54) served by Apache
- MariaDB will be installed on **UBT2** (51.20.129.79)

## Pre-reqs
1. Copy `smkey.pem` to ANS: `~/.ssh/smkey.pem` (permissions `600`).
2. On ANS, clone this repo under `~/ansible-project`.
3. Ensure ansible and necessary Python packages are installed on ANS.
4. Ensure Jenkins can SSH to ANS or that the `smkey.pem` is accessible by Jenkins.

## Commands to run (summary)

On your PC (Git Bash):
```bash
scp -i "/c/Users/Seed Mella/Downloads/smkey.pem" "/c/Users/Seed Mella/Downloads/smkey.pem" ec2-user@16.16.201.56:~/
ssh -i "/c/Users/Seed Mella/Downloads/smkey.pem" ec2-user@16.16.201.56
# then on ANS
mkdir -p ~/ansible-project && cd ~/ansible-project
# clone repo if not already
# install ansible (pip or package manager)
ansible-playbook playbooks/jenkins_install.yml
ansible-playbook playbooks/mariadb.yml
# create artifacts dir
mkdir -p artifacts
```

Then create a Jenkins pipeline job on JNK and point to the Jenkinsfile in this repo. Provide the `smkey.pem` to Jenkins securely (Credentials store) and ensure the key is available at the path configured in `Jenkinsfile` or update the `Jenkinsfile` accordingly.

## Notes
- Replace plaintext DB password and other secrets with Ansible Vault or Jenkins credentials for production.
- Adjust `ansible_user` in the inventory if your AMIs use a different default user (e.g., `ec2-user` vs `ubuntu`).
