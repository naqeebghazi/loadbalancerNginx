# Load balancing with Nginx

This project will demonstrate how to distribute traffic efficiently across multiple servers, optimise performance and ensure high availability for web applications. 


When an application or webpage is hosted on a single computer for people to access, its OK as long as only a few people need access at any given time. What about when hundres, thousands or millions of requests come in to the server? Unless their are replicas of the app/website on other computers, their will be a crash and all users will then be affected. 

Now, lets say you have dozens or even hundreds of servers online to protect yourself from such situations, how do you ensure the workload of user requests is divided up efficiently between the servers. This is where a load balancer comes in and where Nginx can deliver. 

![loadbalancerNginx](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*TrNJZqECEj0eVuJDeNKtNQ.png)

We will use our AWS account to provision three webservers, all running Ubuntu 22.04. Two will become webservers and one will become the nginx load balancer.

![ec2instances](https://github.com/naqeebghazi/loadbalancerNginx/blob/main/images/ec2Instances.png?raw=true)

Each instance must be attached to a security group (SG) with the following settings:

The two Apache webservers SG config:
![](https://github.com/naqeebghazi/loadbalancerNginx/blob/main/images/inboundRulesWebservers.png?raw=true)

The One Nginx Load Balancer SG config:
![](https://github.com/naqeebghazi/loadbalancerNginx/blob/main/images/inboundRulesWebservers.png?raw=true)


Once complete, ssh into each instance into seperate terminals using your IDE (e.g. VSCode):

![](https://github.com/naqeebghazi/loadbalancerNginx/blob/main/images/ec2terminals.png?raw=true)

I have also edited the hostnames of each instance to identify each one better. You can do this by editing the hostname file:

    $ sudo vim /etc/hostname

Suggested names (or your choice):
  Apache1Web
  Apache2Web
  nginxLB

Update the package manager (apt) and install pache on both Apacher servers.

    $ sudo apt update -y &&  sudo apt install apache2 -y

    $   Listen 80
        Listen 8000
        
        <IfModule ssl_module>
                Listen 443
        </Module>
        
        <IfModule mod_gnutls.c>
                Listen 443
        </Module>

Check apache is running on both servers:

    $ sudo systemctl status apache2

Confirmation:

![apache2Installstatus](https://github.com/naqeebghazi/loadbalancerNginx/blob/main/images/apache2status.png?raw=true)

Configure the Apache servers to display a webpage showing the Public IP address:
- Configure web server to serve content on port 8000 (instead of default port 80)
  - Open file:
    
        $ sudo vim /etc/apache2/ports.conf
        
      Now add 'Listen 8000' under 'Listen 80'

![listen8000](https://github.com/naqeebghazi/loadbalancerNginx/blob/main/images/listen8000.png?raw=true)

  - Open file:
  
        $ sudo vi /etc/apache2/sites-available/000-default.conf

![8000virtualHost](https://github.com/naqeebghazi/loadbalancerNginx/blob/main/images/8000VirtualHost.png?raw=true)

Restart Apache to load new config file changes:

    $ sudo systemctl restart apache2

- Create new index.html file. This will contain the code to display the Public IP.
  Create and deit a new index file:

        $ sudo vi index.html

Then copy text below:

                <!DOCTYPE html>
        <html>
        <head>
            <title>My Apache2 EC2 Instance</title>
        </head>
        <body>
            <h1>Welcome to my EC2 instance</h1>
            <p>Public IP: YOUR_PUBLIC_IP</p>
        </body>
        </html>

Ensure 'My Apache1 EC2 Instance' and 'My Apache2 EC2 Instance' are written in the respective instances.

Change file ownership to ensure it executes appropriately:

    $ sudo chown www-data:www-data ./index.html

- Override default html file with our new index file.
    Replace the default html file with our new one above:

        $ sudo cp -f ./index.html /var/www/html/index.html

Then restart the webserver to load new changes:

    $ sudo systemctl restart apache2

Get your public IP and paste this into your browser:

    $ wget -qO- ifconfig.me

Load balancer pages in web browser:

![browserInstancesview](https://github.com/naqeebghazi/loadbalancerNginx/blob/main/images/browserinstances.png?raw=true)

## Configure Nginx as a Load Balancer

Install Nginx on your nginx server before checking its running:

    $ sudo apt update -y && sudo apt install nginx -y
    $ sudo systemctl status nginx

Open the nginx load balancer config file:

    $ sudo vi /etc/nginx/conf.d/loadbalancer.conf

Paste in the following, replacing whats there. then input your Public IP address from the Apache2 and Apache2 servers. 

            
        upstream backend_servers {

            # your are to replace the public IP and Port to that of your webservers
            server 8.130.242.214:8000; # public IP and port for Apache1
            server 3.8.205.144:8000; # public IP and port for Apache2

        }

        server {
            listen 80;
            server_name 18.130.250.11; # Nginx load balancer public IP address

            location / {
                proxy_pass http://backend_servers;
                proxy_set_header Host $host;
                proxy_set_header X-Real-IP $remote_addr;
                proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            }
        }
    
Reload nginx and then test if the file config is ok:

    $ sudo systemctl restart nginx
    $ sudo nginx -t


## Type of Load Balancer Algorithms

1. Round robin
    - Distributes requests sequentially to each server in the pool
    - Best when al servers have similar capabilities/resources
    - Even distribution of traffic
    - Simple to Implement
2. Least connections
    - Least busy server receives newest requests
    - Traffic distributed to least busy server
    - Best when servers have varying capabilities/workloads
3. Weighted round robin
    - Servers assigned different weights based on their capabilities
    - Higher weight, more requests receievd
4. Weighted least connections
    - Servers assigned different weights based on their capabilities
    - Higher weight, more connections receievd
    - Balances traffic based on server capabilities
5. IP Hash
    - Ensures same client always reaches the same server suring a session
    - Best for stateful sessions and maintaining session data

    
