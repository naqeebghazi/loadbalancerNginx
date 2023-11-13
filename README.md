# Load balancing with Nginx

This project will demonstrate how to distribute traffic efficiently across multiple servers, optimise performance and ensure high availability for web applications. 


When an application or webpage is hosted on a single computer for people to access, its OK as long as only a few people need access at any given time. What about when hundres, thousands or millions of requests come in to the server? Unless their are replicas of the app/website on other computers, their will be a crash and all users will then be affected. 

Now, lets say you have dozens or even hundreds of servers online to protect yourself from such situations, how do you ensure the workload of user requests is divided up efficiently between the servers. This is where a load balancer comes in and where Nginx can deliver. 

![loadbalancerNginx](https://miro.medium.com/v2/resize:fit:1400/format:webp/1*TrNJZqECEj0eVuJDeNKtNQ.png)

We will use our AWS account to provision three webservers, all running Ubuntu 22.04:

![ec2instances](https://github.com/naqeebghazi/loadbalancerNginx/blob/main/images/ec2Instances.png?raw=true)


