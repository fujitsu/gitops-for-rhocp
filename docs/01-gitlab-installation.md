# GitLab Installation and Configuration

## Prerequisites

### Example environment

- OS: CentOS-Stream-8-x86_64-20220323
- IP: 192.168.1.101
- RAM: 8 GB
- CPU: 8 cores
- storage: 20G

### Hardware requirements

Please refer to [here](https://docs.gitlab.com/ee/install/requirements.html#hardware-requirements).

## Installation

There are several ways to install GitLab, it is recommended to choose omnibus way to install GitLab, all dependencies have been bundled in the omnibus installation package. Detailed information can be found [here](https://docs.gitlab.com/omnibus).

Following just shows the installation steps for this example.

### Download GitLab community edition RPM package

Visit [here](https://packages.gitlab.com/gitlab/gitlab-ce) to download a gitlab-ce package that suitable for your environment.

For example

```
$ wget --content-disposition https://packages.gitlab.com/gitlab/gitlab-ce/packages/el/8/gitlab-ce-14.8.0-ce.0.el8.x86_64.rpm/download.rpm
```

### Install GitLab

```
$ sudo rpm -ivh gitlab-ce-14.8.0-ce.0.el8.x86_64.rpm
```

### Prepare the configuration file

You can modify /etc/gitlab/gitlab.rb according to your needs. Detailed information can be found [here](https://docs.gitlab.com/omnibus/installation/index.html#installation-and-configuration) and [here](https://docs.gitlab.com/omnibus/#configuring).

In this example, we just use the default configuration, which only uses HTTP service.

### Allow HTTP service in firewall

```
$ sudo firewall-cmd --permanent --add-service=http
$ sudo systemctl reload firewalld
```

### Make the configuration take effect

This step is required regardless of whether the configuration file has been modified or not.

```
$ sudo gitlab-ctl reconfigure
```

### Start GitLab service

```
$ sudo gitlab-ctl start
```

## Configuration

### Add IP Address Resolution for GitLab

Add the GitLab IP address resolution on all the hosts where you perform the access.

```
$ sudo bash -c 'echo "192.168.1.101 gitlab.example.com" >> /etc/hosts'
```

### Login GitLab

Access gitlab.example.com on browser and login admin account:
- Username: root
- Password: See /etc/gitlab/initial_root_password

### Change admin password

1. Click on "Menu" on the top left.
2. Click on "Admin".
3. Click on "Users" under "Overview" on the left.
4. Click on the "Edit" icon on the far right of the "Administrator" row.
5. Enter new password then click on "Save changes" at the bottom. Please note that the password requires no less than 8 characters.

### Turn off the "Anyone can register for an account" for security

1. Click on "Menu" on the top left.
2. Click on "Admin".
3. Click on "General" under "Settings" on the left.
4. Click on "Expand" to expand "Sign-up restrictions".
5. Turn off the "Sign-up enabled" then click on "Save changes" below this area.

### Create accounts for the network and RHOCP administrators

1. Click on "Menu" on the top left.
2. Click on "Admin".
3. Click on "Users" under "Overview" on the left.
4. CLick on "New user" on the right.
5. Enter the required information then click on "Create user" at the bottom.
6. Click on "Edit" on the right of the already created user.
7. Enter initial password then click on "Save changes" at the bottom. Please note that the password requires no less than 8 characters. User will be forced to set the password on first sign in.
