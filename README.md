# Ansible
<b> Ansible is a Configuration Management Tool</b>

## Installation of Ansible
- `sudo dnf update -y `
- `sudo dnf install epel-release -y`
- `sudo dnf install ansible -y`
- `ansible --version`

## Connection between Server(VM1) and Client(VM2) Username-SSH
<b>VM1- Username and Password<br> VM2- SSH Key<br></b>
<b>The Below commands to be executed in Server(VM1)</b>
- Here with command it will generate a public key<br>
`ssh-keygen -t rsa -b 2048`

- To view the content the public key<br>
`cat ~/.ssh/id_rsa.pub`

- To copy the public key to authorized keys<br>
`cat .ssh/id_rsa.pub > .ssh/authorized_keys`

- To view the content of authorized keys<br>
`cat ~/.ssh/authorized_keys`
<img src = "Screenshot 2024-11-17 183925.png" width="700" height="100">

- To view the content of private key<br>
`cat ~/.ssh/id_rsa`

- To see the ip address<br>
`ip a`

- Here with the help of this command you can add ip addresses of VM1 & VM2 to the inventory file(hosts)<br>
`sudo vi /etc/ansible/hosts`

<b>Inventory File</b><br>

<img src = "Screenshot 2024-11-17 170421.png" width="400" height="200">

<b>The below commands to be executed in Client(VM2)</b>
- To see the ip of Client(VM2) which will be added to the inventory file<br>
`ip a`

- To view the authorized keys of Client (if the client vm is created through ssh key then in the authorized_keys there will be already a key generated from azure or if it is created from username and password then the authorized_keys will be blank)<br>
`cat ~/.ssh/authorized_keys`

- If there is already a key generated from azure then don't delete it, add the public key of server(VM1) next to it<br>
`vi ~/.ssh/authorized_keys`

- Then see the authorized_keys again using cat command<br>
`cat ~/.ssh/authorized_keys`
<img src = "Screenshot 2024-11-17 181722.png" width="700" height="300">

### Ping
<b>Ping is being done to see if the connection between 2 machines is being successfully established or not.</b>

- You can use the 1st cmnd if you've provided the name along with the ip in inventory file, if you've only added the ip address then use the 2nd command

- Pinging to the different VM(Server[VM1] -> Client[VM2])<br>
`ansible vm02-client -m ping`<br>
`ansible 10.0.0.5 -m ping`
<img src = "Screenshot 2024-11-17 172842.png" width="700" height="300">

- Pinging to the same VM(Server[VM1] -> Server[VM1])<br>
`ansible vm01-main -m ping`<br>
`ansible 10.0.0.4 -m ping`
<img src = "Screenshot 2024-11-17 173838.png" width="700" height="300">

## Connection between Server(VM1) and Client(VM2) Both SSH
<b>VM1 & VM2 - SSH<br>
In VM1<br></b>
- `sudo chmod 600 .ssh/id_rsa`<br>
- `ls -l .ssh/`<br>

<img src = "Screenshot (759).png" width="700" height="300"><br>

### Ping
<img src = "Screenshot (760).png" width="700" height="300"><br>

## Connection between Server(VM1) and Client(VM2) Both Username and Password

<b> Do the following things on Server(VM1)</b><br>
- `ssh-keygen -t rsa -b 2048`<br>
- `cat .ssh/id_rsa.pub`<br>
- `cat .ssh/id_rsa.pub >> .ssh/authorized_keys`<br>
Add the ip addresses in the inventory file<br>
- `sudo vi /etc/ansible/hosts`<br>

<b> Do the following things on Client(VM2)</b><br>
- Add the public key from VM1(Server) to VM2(Client) authorized keys


## AD-HOC Commands
- `ansible client -m command -a "sleep 120"`<br>

<img src = "Screenshot (762).png" width="700" height="300"><br>
- `ansible client -m command -a "sudo dnf install git -y"`<br>

<img src = "Screenshot (763).png" width="700" height="300">

## Ansible Playbook
<b>installations.yaml</b><br>
```yaml
---
- name: Install Maven and Other Dependencies
  hosts: server
  become: true
  tasks:
    - name: Install Maven
      dnf:
        name: maven
        state: present

    - name: Remove Podman and Podman-Docker to Avoid Docker Conflict
      dnf:
        name:
          - podman-docker
          - podman
        state: absent
      ignore_errors: yes

    - name: Download the Latest kubectl Binary
      shell: |
        curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
      args:
        creates: /usr/local/bin/kubectl

    # Additional tasks ...

    - name: Install Azure CLI
      dnf:
        name: azure-cli
        state: present

    - name: Verify Azure CLI Installation
      shell: az --version
      register: azure_cli_version_output
      changed_when: false

    - debug:
        var: azure_cli_version_output.stdout

- name: Install Terraform
  hosts: server
  become: true
  tasks:
    - name: Install yum-utils
      yum:
        name: yum-utils
        state: present

    - name: Add HashiCorp Repository
      command: >
        yum-config-manager --add-repo https://rpm.releases.hashicorp.com/RHEL/hashicorp.repo
      args:
        creates: /etc/yum.repos.d/hashicorp.repo

    - name: Install Terraform
      yum:
        name: terraform
        state: present

    - name: Verify Terraform Installation
      command: terraform --version
      register: terraform_version_output
      ignore_errors: no

    - name: Display Terraform Version
      debug:
        msg: "Installed Terraform version: {{ terraform_version_output.stdout }}"

