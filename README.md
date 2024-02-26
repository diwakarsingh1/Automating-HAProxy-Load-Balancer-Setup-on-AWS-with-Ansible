# Automating-HAProxy-Load-Balancer-Setup-on-AWS-with-Ansible
# About the Project -

This project implements a full automation of Load-Balancing setup using HAProxy on AWS with the help of Ansible Playbook.

![](/ansible_automation/Banner.webp)

# Step 1: Launch 5 AWS EC2 instances.
One instance for Ansible Controller. One for Load Balancer (Frontend). And rest three for Web Server (Backend).

# Step 2: Allow the Frontend & Backend instances for ssh remote login.
Set the root password with the help of:

    passwd root

Now go to the location.
    
    cd /etc/ssh

You will see different files and directories, as shown below:

![](/ansible_automation/ssh-ls.webp)

Now edit the file name sshd_config using:

        vim sshd_config

Now you have to do ***[ yes ]*** to the 2 of the values one is ***PermitRootLogin*** and ***PasswordAuthentication***, after doing it save the file by pressing ***[ esc ]*** and then typing ***[ :wq ]*** and press ***[ enter ]***.

![](/ansible_automation/PermitRootLogIn.webp)
![](/ansible_automation/PasswordAuthentication.webp)

After doing it restart the sshd service with the help of:

    systemctl restart sshd

Now, do the ssh from ansible controller instance to all the frontend and backend instance. Using the command below. In place of ***{ ip_address }*** you have to use the ip_address of the instance where you want to connect using ssh. Be careful of doing the ssh to all the instances.

    ssh -l root ip_address

# Step 3: Create the inventory in ansible controller instance.
Now open the inventory file using the command below:

    vim /etc/ansible/hosts

Now add the ip_address of the frontend and backend along with username, password, protocol name using the syntax given below, by creating two different groups. Let says for Frontend the group name is ***[ LoadBalancer ]*** and Backend ***[ WebServer ]***.

    [ LoadBalancer ]
    ip_address_frontend ansible_user=root ansible_password=Your_Password ansible_connection=ssh

    [ WebServer ]
    ip_address_backend_1 ansible_user=root ansible_password=Your_Password ansible_connection=ssh
    ip_address_backend_2 ansible_user=root ansible_password=Your_Password ansible_connection=ssh
    ip_address_backend_3 ansible_user=root ansible_password=Your_Password ansible_connection=ssh

Now Ping to all the 4 instances from the Ansible Controller instance inorder to check the connection between controller and ***[ Frontend & Backend ]***. If it is success it will show the SUCCESS as given below.

![](/ansible_automation/success.webp)

# NOTE:- Now all the operation will be performed in Ansible Controller Instance.
# Step 4: Now set-up the Ansible Playbook for Frontend instance.

Create the file name haproxy.j2 ***[ j2 is for Jinja as we are using Jinja for templating in Ansible ]***.

    vim haproxy.j2

Now write the following code for haproxy configuration.

    frontend LB
        bind *:8080    
        timeout client 10s
        default_backend webserver

    backend webserver
        balance roundrobin
        timeout server 10s
        timeout connect 10s
    {% for ip in groups.WebServer %}
        server app{{ loop.index }} {{ ip }}:80
    {% endfor %}

<h4> Explanation of above code. </h4>
1. Frontend Configuration:
    — Binds to port 8080 on all interfaces (`bind *:8080`).
    — Sets a client timeout of 10 seconds (`timeout client 10s`).
    — Routes incoming requests to the `webserver` backend (`default_backend webserver`).
2. Backend Configuration:
    — Defines the `webserver` backend.
    — Uses round-robin load balancing (`balance roundrobin`).
    — Sets a server timeout of 10 seconds (`timeout server 10s`).
    — Sets a connection timeout of 10 seconds (`timeout connect 10s`).

3. Server Configuration (Loop):
    — Iterates over each server IP in the `WebServer` group.
    — Configures a server entry for each IP, incrementally numbered (`server app{{ loop.index }} {{ ip }}:80`).
