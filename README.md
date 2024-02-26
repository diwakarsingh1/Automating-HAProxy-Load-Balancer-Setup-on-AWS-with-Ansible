# Automating-HAProxy-Load-Balancer-Setup-on-AWS-with-Ansible
# About the Project -

This project implements a full automation of Load-Balancing setup using HAProxy on AWS with the help of Ansible Playbook.

![](/ansible_automation/Banner.webp)

# Step 1: Launch 5 AWS EC2 instances.
One instance for Ansible Controller. One for Load Balancer (Frontend). And rest three for Web Server (Backend).

# Step 2: Allow the Frontend & Backend instances for ssh remote login.
Set the root password with the help of:
# code
    passwd root

