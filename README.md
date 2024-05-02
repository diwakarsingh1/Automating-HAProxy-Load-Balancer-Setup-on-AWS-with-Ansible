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

<h2> Explanation of above code. </h2>

<h3> 1. Frontend Configuration: </h3>

- __Binds to port 8080 on all interfaces (`bind *:8080`).__

- __Sets a client timeout of 10 seconds (`timeout client 10s`).__
  
- __Routes incoming requests to the `webserver` backend (`default_backend webserver`).__

<h3>  2. Backend Configuration: </h3>

- __Defines the `webserver` backend.__
  
- __Uses round-robin load balancing (`balance roundrobin`).__
  
- __Sets a server timeout of 10 seconds (`timeout server 10s`).__
  
- __Sets a connection timeout of 10 seconds (`timeout connect 10s`).__

<h3>  3. Server Configuration (Loop): </h3>

- __Iterates over each server IP in the `WebServer` group.__
  
- __Configures a server entry for each IP, incrementally numbered (`server app{{ loop.index }} {{ ip }}:80`).__

Now create the file name __LB__ with extension ***[ .yml ]***. Using:

    vim LB.yml

And type the below code:

    - hosts: LoadBalancer
      tasks:
        - name: Installing haproxy
          package:
            name: "haproxy"
            state: present
        - name: Registering Webserver TO LB
          template:
            src: "haproxy.j2"
            dest: "/etc/haproxy/haproxy.cfg"

        - name: Starting haproxy service
          service:
            name: "haproxy"
            state: restarted


<h2> Explanation of above code. </h2>

<h3> 1. Installing HAProxy: </h3>

- __Uses the Ansible `package` module to ensure HAProxy is present on the Load Balancer hosts.__

<h3> 2. Registering Webserver to Load Balancer: </h3>

- __Copies the HAProxy configuration template file (`haproxy.j2`) to `/etc/haproxy/haproxy.cfg` on the Load Balancer hosts.__

<h3> 3. Starting HAProxy Service: </h3>

- __Uses the Ansible `service` module to restart the HAProxy service on the Load Balancer hosts, applying the new configuration.__

Now run the LB.yml playbook using:

    ansible-playbook LB.yml

![](/ansible_automation/playbook.webp)

# Step 5: Now set-up the Ansible Playbook for Backend instance.

Create the file name index.php using:

    vim index.php

And paste the given php code.

    <pre>
    <?php
      print_r($_SERVER);
    ?>
    </pre>

This code is simple you will see the server side details its ip_address and all necessary server side details.

![](/ansible_automation/sample-php.webp)

Now create the playbook for configuring all the backend instances. Let says the file is web.yml

    - hosts: WebServer
      tasks:
        - name: Installing HTTPD
          package:
            name: "httpd"
            state: present

        - name: Installing PHP
          package:
            name: "php"
            state: present

        - name: Copying Php file to default location
          template:
            src: "index.php"
            dest: "/var/www/html"

        - name: Starting HTTPD service
          service:
            name: "httpd"
            state: started

<h2> Explanation of above code. </h2>

<h3> 1. Installing HTTPD: </h3>

- __Uses the Ansible `package` module to ensure the Apache HTTP Server (`httpd`) is installed on the hosts in the WebServer group.__

<h3> 2. Installing PHP: </h3>

- __Uses the Ansible `package` module to ensure PHP is installed on the hosts in the WebServer group.__

<h3> 3. Copying PHP file to default location: </h3>
   
- __Copies the `index.php` file to the `/var/www/html` directory on the hosts in the WebServer group using the Ansible `template` module.__

<h3> 4. Starting HTTPD service: </h3>

- __Uses the Ansible `service` module to start the Apache HTTP Server (`httpd`) service on the hosts in the WebServer group.__

Now run the web.yml playbook using:

    ansible-playbook web.yml

Congratulation you have set-up HAProxy Load Balancer on AWS with Ansible.
Now type the ip_address of your Frontend using port number that you have used in haproxy.j2 file.
__ip_address:8080__

Let me show you the example of frontend getting connected to different backend instances whenever client try to reach frontend IP.

![](/ansible_automation/backend-1.webp)
***Here it connected to [ 172.31.35.49 ] Backend.***

![](/ansible_automation/backend-1.webp)
***Here it connected to [ 172.31.47.91 ] Backend.***

![](/ansible_automation/backend-1.webp)
***Here it connected to [ 172.31.36.67 ] Backend***

