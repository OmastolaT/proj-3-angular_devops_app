# Demo: Angular Static Site + Jenkins + Ansible + MariaDB

## Overview
- Ansible controller runs on **ANS** (16.16.201.56)
- Jenkins on **JNK** (13.60.9.2)
- Angular site deployed on **UBT1** (13.53.37.54)
- MariaDB on **UBT2** (51.20.129.79)

## Steps
1. Copy `smkey.pem` to ANS `~/.ssh/smkey.pem`
2. SSH into ANS, set permissions, install ansible
3. Run playbooks:
   ```bash
   ansible-playbook playbooks/jenkins_install.yml
   ansible-playbook playbooks/mariadb.yml
   ```
4. Jenkins will handle build and deploy via Jenkinsfile.

Open app at http://13.53.37.54
