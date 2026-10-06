```
// NACL 

1. create ACL (my-acl) in default-vpc
2. associate my-acl to subnet subnet-us-east-1a
3. create inbound rule (ACL)
100 | http | 80 | 0.0.0.0/0 | allow
101 | http | 22 | 0.0.0.0/0 | allow
4. create outbound rule (ACL)
100 | custom-tcp | 1024-65535 | 0.0.0.0/0 | allow
5. launch and connect instance in. subnet-us-east-1a
(SG) - 22, 80 
6. 
sudo yum install httpd -y 
sudo yum update -y
sudo yum install httpd -y 
sudo systemctl start httpd
sudo systemctl enable httpd 
cd /var/www/html
sudo chmod 755 /var/www/html/
sudo touch index.html
sudo nano index.html 

<h1> Webserver </h1>

flow: laptop >> vpc >> (ACL) subnent >> nic (SG) >> instance (webpage) 

7. check public ip of instance in browser 

http://instance-public-ip

8. 
diassociate subnet from acl
delete acl 
terminate instance 
delete keypair
delete SG 
```
