# LOAD BALANCER SOLUTION WITH APACHE

Modern high‑traffic websites must serve hundreds of thousands, if not millions, of concurrent requests from users or clients and return the correct text, images, video, or application data, all in a fast and reliable manner. To cost‑effectively scale to meet these high volumes, modern computing best practice generally requires adding more servers.

A load balancer acts as the “traffic cop” sitting in front of your servers and routing client requests across all servers capable of fulfilling those requests in a manner that maximizes speed and capacity utilization and ensures that no one server is overworked, which could degrade performance. If a single server goes down, the load balancer redirects traffic to the remaining online servers. When a new server is added to the server group, the load balancer automatically starts to send requests to it.

A load balancer performs the following functions:

* Distributes client requests or network load efficiently across multiple servers
* Ensures high availability and reliability by sending requests only to servers that are online
* Provides the flexibility to add or subtract servers as demand dictates

 ### Aim:

  In this project we will enhance our Tooling Website solution (project7) by adding a Load Balancer to disctribute traffic between Web Servers and allow users to access our website using a single URL.

### Task
  
  Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 intance. Make sure that users can be served by Web servers through the Load Balancer.

 ### Steps:
CONFIGURE APACHE AS A LOAD BALANCER

1. Create an Ubuntu Server 20.04 EC2 instance and name it Project-8-apache-lb

2. We will open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in Security Group.

3. We will install Apache Load Balancer on Project-8-apache-lb server and configure it to point traffic coming to LB to both Web Servers:

```
#Install apache2
sudo apt update
sudo apt install apache2 -y
sudo apt-get install libxml2-dev

#Enable following modules:
sudo a2enmod rewrite
sudo a2enmod proxy
sudo a2enmod proxy_balancer
sudo a2enmod proxy_http
sudo a2enmod headers
sudo a2enmod lbmethod_bytraffic

#Restart apache2 service
sudo systemctl restart apache2
```

After apache2 successful installation and enabling all necessary modules the image below shows our apache load balancer is up and running.

![Screenshot 2023-06-21 053318](https://github.com/opeyemiogungbe/Darey.io-PBL-PROJECT/assets/136735745/7feff130-55f1-47fe-8ed7-e3cc2f877021)


**Configure load balancing**

```
sudo vi /etc/apache2/sites-available/000-default.conf

#Add this configuration into this section <VirtualHost *:80>  </VirtualHost>

<Proxy "balancer://mycluster">
               BalancerMember http://<WebServer1-Private-IP-Address>:80 loadfactor=5 timeout=1
               BalancerMember http://<WebServer2-Private-IP-Address>:80 loadfactor=5 timeout=1
               ProxySet lbmethod=bytraffic
               # ProxySet lbmethod=byrequests
        </Proxy>

        ProxyPreserveHost On
        ProxyPass / balancer://mycluster/
        ProxyPassReverse / balancer://mycluster/

#Restart apache server

sudo systemctl restart apache2
```

![Screenshot 2023-08-29 052041](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/096f6aec-3f1f-4d20-a44b-6c37ee8b3bf6)

The above image shows our load balancer configuration method which is bytraffic balancing method. This method will distribute incoming load between our Web Servers according to current traffic load. We can control in which proportion the traffic must be distributed by loadfactor parameter. In thhe settings above our loadfactor parameters is set to 5 traffic request each. We allso have other load balancing method like: bybusyness, byrequests, heartbeat.


Next we will verify that our configuration works – try to access your LB’s public IP address or Public DNS name from your browser:

http://<Load-Balancer-Public-IP-Address-or-Public-DNS-Name>/index.php

![Screenshot 2023-08-29 053214](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/3a54eea2-b742-498a-b4c8-031723d9bffd)

The image above is the result of our load balancer directing traffic to our tooling website from project7

**Optional Step – Configure Local DNS Names Resolution**

We can also choose to configure local domain name resolution to help us remember and switch between IP addresses using /etc/hosts file, which is very easy to configure but not very scalable.

```
#Open this file on your LB server

sudo vi /etc/hosts

#Add 2 records into this file with Local IP address and arbitrary name for both of your Web Servers

<WebServer1-Private-IP-Address> Web1
<WebServer2-Private-IP-Address> Web2
```

![Screenshot 2023-08-29 061057](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/0e5e3f25-d341-4752-8f79-58090463298d)

Now you can update your LB config file with those names instead of IP addresses.

```
BalancerMember http://Web1:80 loadfactor=5 timeout=1
BalancerMember http://Web2:80 loadfactor=5 timeout=1
```

Now we will try to curl our Web Servers from LB locally curl http://Web1 or curl http://Web2 – it shall work.


![Screenshot 2023-08-29 072008](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/c4ce2300-7536-4972-a6e3-5d882bfe7b41)


![Screenshot 2023-08-29 072145](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/6bf9aa69-ec37-4084-b336-637c307ba405)

Yes! it worked now our architecture settings look like the image below:

![Screenshot 2023-11-05 171346](https://github.com/opeyemiogungbe/Pbl-project7/assets/136735745/12d09110-5e49-481e-ab76-8e6901b404da)

we are done with our load balancing project.
