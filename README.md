# Installing and Running Ansible Locally

## Overview

This section provides a step-by-step guide to install and run Ansible locally on a Windows system using the Windows Subsystem for Linux (WSL) with Ubuntu 22.04. This setup is useful for development and testing of Ansible playbooks in a local environment.

### Note
If you are running this from a Hyper-V Virtual Machine you must enable nested virtualization
`Set-VMProcessor -VMName '<vmname>' -ExposeVirtualizationExtensions $True`

## Steps

1. **Install WSL with Ubuntu 22.04**:
   - Run `wsl --install -d Ubuntu-22.04` in the Windows command prompt.
   - Follow the instructions to set up Ubuntu, including entering a username and password.

2. **Update and Upgrade Ubuntu Packages**:
   ```bash
   sudo apt update
   sudo apt upgrade
   ```

3. **Install Python and Dependencies**:
   - Set the environment to non-interactive to prevent package install prompts:
     ```bash
     export DEBIAN_FRONTEND=noninteractive
     ```
   - Install Python, Kerberos support, and other required packages:
     ```bash
     sudo -E apt install -y python3-pip python3-kerberos python3-dev libkrb5-dev krb5-user iputils-ping
     ```

4. **Install Ansible and pywinrm**:
   - Install Ansible using Python pip:
     ```bash
     python3 -m pip install --user ansible
     ```
   - Install the Python Windows Remote Management (pywinrm) package for Kerberos support:
     ```bash
     pip install pywinrm[kerberos]
     ```
   - Update the `.bashrc` file to include the local bin directory in the PATH:
     ```bash
     echo "export PATH=$PATH:/home/$USER/.local/bin" >> ~/.bashrc
     ```

5. **Configure WSL Network Settings**:
   - Disable automatic generation of `resolv.conf`:
     ```bash
     sudo tee -a /etc/wsl.conf <<'EOF'
     [network]
     generateResolvConf = false
     EOF
     ```
   - Exit the WSL environment and shut it down and restart:
     ```bash
     exit
     wsl --shutdown
     wsl -d Ubuntu-22.04
     ```
   - Set custom DNS servers in `resolv.conf`:
     ```bash
     sudo tee -a /etc/resolv.conf <<'EOF'
     nameserver x.x.x.x
     nameserver x.x.x.x
     EOF
     ```
   - Set Kerberos realm in `krb5.conf`:
     ```bash
     sudo tee -a /etc/krb5.conf <<'EOF'
     [libdefaults]
       rdns = false
       default_realm = AD.DOMAIN.LOCAL
         dns_lookup_realm = true
         dns_lookup_kdc = true
     [realms]
     AD.DOMAIN.LOCAL = {
       kdc = DC.ad.domain.local
       admin_server = DC.ad.domain.local
     }
     EOF
     ```

6. **Run Ansible Playbook**:
   - Execute the Ansible playbook by specifying the inventory file and playbook path. The `--extra-vars` option can be used to pass additional variables:

     ```bash
     ansible-playbook -i hosts.yml ./playbooks/maintenance/servers/vbridge/vbridge_snapshot.yml --extra-vars "@secrets.yml"
     ```

     ```bash
     ansible-playbook -i hosts.yml ./playbooks/maintenance/servers/patch_servers.yml --extra-vars "@secrets.yml"
     ```