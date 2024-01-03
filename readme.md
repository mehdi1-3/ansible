

# Kubernetes Cluster Setup with Ansible

This repository contains Ansible playbooks for automating the setup of a Kubernetes cluster. The playbooks handle the configuration of the Kubernetes master and worker nodes.

## Prerequisites

Before running the playbooks, an SSH key pair should be generated and the ssh-agent should be started. The ssh-agent is a program that holds private keys used for public key authentication (RSA, DSA, ECDSA, Ed25519). The ssh-add command is used to add private key identities to the authentication agent.

To generate an SSH key pair and start the ssh-agent, run the following commands:

```bash
ssh-keygen -t rsa -b 2048 -f ~/.ssh/id_rsa
eval `ssh-agent`
ssh-add ~/.ssh/id_rsa
```

To ensure the ssh-agent starts and the keys are added every time a new terminal session is opened, add the following lines to your shell's profile script (`.bashrc`, `.bash_profile`, or `.zshrc` for Bash or Zsh):

```bash
if ! pgrep -q -u "$USER" ssh-agent; then
    ssh-agent > ~/.ssh-agent-env
fi

if [[ -f ~/.ssh-agent-env ]]; then
    . ~/.ssh-agent-env
    if ! ssh-add -l >/dev/null; then
        ssh-add ~/.ssh/cluster
    fi
fi
```

## Playbooks

### newUser.yaml

This playbook creates a new user on all nodes, sets up sudo without a password for the new user, sets an authorized key for the new user, removes Docker and containerd packages and directories, and updates the system.

Usage:

```bash
ansible-playbook newUser.yaml
```

### install.yaml

This playbook updates hostnames, disables swap, configures containerd, adds Docker and Kubernetes repositories, installs Kubernetes cluster packages, enables the kubelet service, and reboots all the Kubernetes nodes.

Usage:

```bash
ansible-playbook install.yaml
```

### masterConf.yaml

This playbook sets up the master node. It pulls the required images, resets and initializes the Kubernetes cluster, copies the `admin.conf` file to the local Ansible server, gets the token for joining worker nodes, installs the pod network, and copies various outputs to the local Ansible server.

Usage:

```bash
ansible-playbook -i hosts masterConf.yaml
```

### workersConf.yaml

This playbook sets up the worker nodes. It copies the join command token to the worker nodes, resets kubeadm, and joins the worker nodes to the master.

Usage:

```bash
ansible-playbook -i hosts workersConf.yaml
```

### install_kubectl.yaml

This playbook installs `kubectl` on the Ansible VM. It updates the package list, installs the necessary prerequisites, creates the `/etc/apt/keyrings` directory if it doesn't exist, adds the Kubernetes key, updates the package list again, installs `kubectl`, and verifies that `kubectl` is working.

Usage:

```bash
ansible-playbook install_kubectl.yaml
```

## Inventory File

The `hosts` file is an inventory file for Ansible. It lists the master and worker nodes that the playbooks will operate on.

## Note

Before running the playbooks, make sure to update the `hosts` file with the correct IP addresses or hostnames of your nodes. Also, replace any placeholder usernames and other values in the playbooks with your actual values.
```

This version of the `README.md` file provides a more detailed explanation of the prerequisites and the purpose of each playbook. It also includes usage instructions for each playbook.