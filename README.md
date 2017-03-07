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

### Install Finger
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
ssh -i ~/.ssh/grader_key.rsa grader@52.91.39.237
```
When you can log on, modify the permissions 
```
chmod 700 .ssh
chmod 644 .ssh/authorized_keys
```
