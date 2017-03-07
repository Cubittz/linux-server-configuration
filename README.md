# linux-server-configuration
Udacity Full Stack Nanodegree project to configure a linux web server. This readme file explains some of the settings and configuration options that were chosed when building the server.

### General Settings
Public IP: 52.91.39.237

Username: ubuntu

Port: 2200

### Run updates

```
$ sudo apt-get update
$ sudo apt-get upgrade
```
### Change ssh port to 2200
Create Custom rule in Lightsail firewall to allow TCP on port 2200.

Run following command to open sshd_config and look for line specifying which ports to list for. Change 22 to 2200
```
$ sudo nano /etc/ssh/sshd_config
```
### Install Finger (optional)
Install finger to see additional information for users
```
$ sudo apt-get install finger
```
### Add grader User
Create a new user called grader to allow Udacity reviewer to login
```
$ sudo adduser grader
```
Then grant sudo access  - create a grader file in /etc/sudoers.d with the same content as the default ubuntu one

### Generate key pair
On host machine, generate a new key pair and save file in /Users/*username*/.ssh/grader_key using
```
$ ssh-keygen
```
Copy content of ```.pub``` file to ```/home/grader/.ssh/authorized_keys``` and then log on as grader user
```
$ ssh -i ~/.ssh/grader_key grader@52.91.39.237
```
When you can log on, modify the permissions 
```
$ chmod 700 .ssh
$ chmod 644 .ssh/authorized_keys
```
### Force Key Based Authentication
To ensure users can only log in with a key, run the following command (already configured on Lightsail)
```
$ sudo nano /etc/ssh/sshd_config
```
And change ```PasswordAuthentication``` to ```no```. Once changed, run the following command so the server restarts for it to take effect
```
$ sudo service ssh restart
```
