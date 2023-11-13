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

- Create new index.html file. This will contain the code to dsiaply the Public IP.
- Override default html file with our new index file. 
