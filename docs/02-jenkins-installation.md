# Jenkins Installation and Configuration

## Prerequisites

### Example environment

- OS: CentOS-Stream-8-x86_64-20220323
- IP: 192.168.1.102
- RAM: 4 GB
- CPU: 4 cores
- storage: 20G

### Hardware requirements

Please refer to [here](https://www.jenkins.io/doc/book/installing/linux/#prerequisites).

## Installation

Detailed information can be found [here](https://www.jenkins.io/doc/book/installing/linux).

Following just shows the installation steps for this example.

### Install dependencies

```
$ sudo dnf install java-11-openjdk git -y
```

### Install Jenkins

```
$ sudo wget -O /etc/yum.repos.d/jenkins.repo https://pkg.jenkins.io/redhat-stable/jenkins.repo
$ sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key
$ sudo yum upgrade
$ sudo yum install jenkins -y
$ sudo systemctl daemon-reload
```

### Prepare the configuration file

You can modify /etc/sysconfig/jenkins according to your needs. Items that can be configured are listed in this file.

In this example, we just use the default configuration, which the port is 8080.

### Allow access to port 8080 in the firewall

```
$ sudo firewall-cmd --permanent --add-port=8080/tcp
$ sudo firewall-cmd --reload
```

### Start Jenkins service

```
$ sudo systemctl start jenkins
$ sudo systemctl enable jenkins
```

## Configuration

### Add IP Address Resolution for Jenkins

Add the Jenkins IP address resolution on all the hosts where you perform the access.

```
$ sudo bash -c 'echo "192.168.1.102 jenkins.example.com" >> /etc/hosts'
```

### Login Jenkins

Access jenkins.example.com:8080 on browser, then the first step of the post-installation setup wizard will show up. Detailed information can be found [here](https://www.jenkins.io/doc/book/installing/linux/#unlocking-jenkins).

The Jenkins initial admin password is as follows:

```
$ sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

### Install plugins

This is the second step of the post-installation setup wizard. Detailed information can be found [here](https://www.jenkins.io/doc/book/installing/linux/#customizing-jenkins-with-plugins).

Click on "Select plugins to install" in the setup wizard, add following extra plugins to Jenkins.

- GitLab
- Git Parameter

### Create user

This is the last step of the post-installation setup wizard. Detailed information can be found [here](https://www.jenkins.io/doc/book/installing/linux/#creating-the-first-administrator-user).

In this example, we just skip this step.

### Change admin password

1. Click on "Dashboard" on the top left.
2. Click on "Manage Jenkins".
3. Click on "Manage Users" under "Security".
4. Click on the gearwheel icon on the far right of the "admin" row.
5. Enter new password then click on "Apply" at the bottom.

### Create accounts for the network and RHOCP administrators

1. Click on "Dashboard" on the top left.
2. Click on "Manage Jenkins".
3. Click on "Manage Users" under "Security".
4. Click on "Create User" on the left.
5. Enter the user information then click on "Create User".
