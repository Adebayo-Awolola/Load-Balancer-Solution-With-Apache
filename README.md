# Load-Balancer-Solution-With-Apache

A Load Balancer (LB) distributes clients' requests among underlying Servers and ensures the load is distributed optimally.

The diagram below shows the architecture of the solution

![image](https://github.com/user-attachments/assets/a5821926-3702-4f6e-bbf2-1ce1db494f67)

Task
Deploy and configure an Apache Load Balancer for Tooling Website solution on a separate Ubuntu EC2 instance. Make sure that users can be served by Web servers through the Load Balancer.

 Prerequisites

Ensure that the following servers are installed and configured already.

- Two RHEL9 Web Servers
- One MySQL DB Server (based on Ubuntu 24.04)
- One RHEL9 NFS Server

 Prerequisites Configurations

- Apache (httpd) is up and running on both Web Servers.
- '/var/www' directories of both Web Servers are mounted to '/mnt/apps' of the NFS Server.
- All neccessary TCP/UDP ports are opened on Web, DB and NFS Servers.
- Client browsers can access both Web Servers by their Public IP addresses or Public DNS names and can open the 'Tooling Website' (e.g, 'http://<Public-IP-Address-or-Public-DNS-Name>/index.php')

 Step 1 - Configure Apache As A Load Balancer
 1. Create an Ubuntu Server 24.04 EC2 instance and name it Project-8-apache-lb
    ![image](https://github.com/user-attachments/assets/5a6a80e8-44e5-4505-a396-d28eb71a920d)

 3. Open TCP port 80 on Project-8-apache-lb by creating an Inbound Rule in the Security Group
    ![image](https://github.com/user-attachments/assets/adb258e4-705f-4224-b42f-85f6ec27140d)

 5. Install Apache Load Balancer on Project-8-apache-lb and configure it to point traffic coming to LB to both Web Servers.
 i. Install Apache2
- Access the instance
ssh -i "MEAN.pem" ubuntu@ec2-54-145-97-90.compute-1.amazonaws.com
'- Update and upgrade Ubuntu
-sudo apt update && sudo apt upgrade -y
![image](https://github.com/user-attachments/assets/4cb4eded-ac23-4073-9dd0-324a1ed9181a)

- Install Apache
- sudo apt install apache2 -y
'
![image](https://github.com/user-attachments/assets/112ea0b4-7d67-4920-ad52-52ed032c82a7)
sudo apt-get install libxml2-dev -y
![image](https://github.com/user-attachments/assets/88a14643-81a3-49bf-9aaf-dbcc64892c6a)

ii. Enable the following modules

'
sudo a2enmod rewrite

sudo a2enmod  proxy

sudo a2enmod  proxy_balancer

sudo a2enmod  proxy_http

sudo a2enmod  headers

sudo a2enmod  lbmethod_bytraffic
'
![image](https://github.com/user-attachments/assets/d14bd031-931a-4265-a298-0c20e5a2ed13)

iii. Restart Apache2 Service
sudo systemctl restart apache2
sudo systemctl status apache2
"![image](https://github.com/user-attachments/assets/9021eca3-0697-49d1-aeae-bb05ef1754dd)"


 Configure Load Balancing

 i. Open the file 000-default.conf in sites-available

'
sudo vi /etc/apache2/sites-available/000-default.conf
'
 ii. Add this configuration into the section '<VirtualHost *:80>  </VirtualHost>'

'apache
<Proxy "balancer://mycluster">
            BalancerMember http://18.209.173.19:80 loadfactor=5 timeout=1
           BalancerMember http://54.162.250.21:80 loadfactor=5 timeout=1
           ProxySet lbmethod=bytraffic
</Proxy>


ProxyPreserveHost on
ProxyPass / balancer://mycluster/
ProxyPassReverse / balancer://mycluster/
'


 iii. Restart Apache
sudo systemctl restart apache2

'bytraffic' balancing method with distributing incoming loads between the Web Servers according to current traffic load. The proportion in which traffic must be distributed can be controlled by 'loadfactor' parameter.

Other methods such as 'bybusyness', 'byrequests', 'heartbeat' can also be adopted.
4. Verify that the configuration works

 i. Access the website using the LB's Public IP address or the Public DNS name from a browser

"![image](https://github.com/user-attachments/assets/d6f5219f-89ca-465f-9196-6af7995f5c95)"

Note_: If in the previous project, '/var/log/httpd' was mounted from the Web Server to the NFS Server, unmount them and ensure that each Web Servers has its own log directory.

 ii. Unmount the NFS directory

- Check if the Web Server's log directory is mounted to NSF
df -h
sudo umount -f /var/log/httpd
![image](https://github.com/user-attachments/assets/0ea24421-5cca-47f4-b9b5-494ecf04a149)

- Check that the directory is unmounted
'
df -h
' iii. Open two ssh consoles for both Web Server and run the command:
sudo tail -f /var/log/httpd/access_log
Web Server 1 & 2 'access_log'
 Weserver 1

![image](https://github.com/user-attachments/assets/a5feee3c-e4d9-4956-967d-87bf4c968e30)
Webserver 2
"![image](https://github.com/user-attachments/assets/ac7d4d47-eeae-4450-ae61-def858b8e82b)"

 iv. Refresh the browser page several times and ensure both Web Servers receive HTTP and GET requests. New records must apear in each web server log files. The number of request to each servers will be approximately the same since 'loadfactor' is set to the same value for both servers. This means that traffic will be evenly distributed between them.
 After the refresh.
"![image](https://github.com/user-attachments/assets/f20e06c3-04d3-4b87-b2e5-5684838cfa98)"

Sometimes, remembering and switching between IP addresses can be tedious, especially if there are many servers to manage. It is best to configure local domain name resolution. The easiest way is to use the'/etc/hosts' file. Although this approach is not very scalable, it is very easy to configure and shows the concept well.

 Configure the IP address for domain name mapping for our load balancer.

 Open the hosts file on your lb server
sudo vi /etc/hosts
' Add two records into file with Local IP address and arbitrary name for the Web Servers
"![image](https://github.com/user-attachments/assets/9e5e4591-5d86-4c4f-b1f1-96ebcc85b054)"

 Update the LB config file with those arbitrary names instead of IP addresses on the LB server.

sudo vi /etc/apache2/sites-available/000-default.conf
'
'
BalancerMember http://web1:80 loadfactor=5 timeout=1
BalancerMember http://web2:80 loadfactor=5 timeout=1
'
"![image](https://github.com/user-attachments/assets/986d9b01-f210-4c1a-a29f-a37a3f31dd8f)"

 Try to curl the Web Servers from LB locally
curl http://web1
"![image](https://github.com/user-attachments/assets/8b4883b5-3179-4a80-8bac-af9a77c20375)"

curl http://web2
"![image](https://github.com/user-attachments/assets/4135da78-e38b-4388-84bc-1afac2e47282)"
Remember, This is only internal configuration and also local to the LB server, these names will neither be 'resolvable' from other servers internally nor from the Internet.
 
Now your set up looks like this:
"![image](https://github.com/user-attachments/assets/8b235db6-2b6a-4640-a7e5-813b4c3ab44b)"



