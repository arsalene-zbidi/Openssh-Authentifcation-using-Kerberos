
# Openssh Authentifcation using Kerberos 

This is a demo of how to authentifcate to openssh connectivity tool using **Gasspi** and **Kerberos** in order to allow **single-sign-on** Authentifcation without password.

# Kerberos 
Kerberos is a widely used network authentication protocol that provides secure authentication and authorization for clients and servers in a distributed computing environment. It uses a system of tickets to verify identities and supports mutual authentication between clients and servers.

## Kerberos Architecture
The Kerberos architecture consists of three main components: the client, the server, and the Kerberos authentication server (KDC). The KDC is responsible for issuing and verifying tickets, while the client and server use these tickets to authenticate and access network resources securely. The protocol uses symmetric-key cryptography and mutual authentication to ensure secure identity verification in distributed systems.
# Openssh
OpenSSH is a free and open-source implementation of the Secure Shell (SSH) protocol used to securely access and manage remote servers and devices. It provides encrypted communication between client and server, enabling secure data transfer and remote command execution. OpenSSH includes a suite of tools, including ssh, scp, and sftp, which allow users to securely connect to remote servers, transfer files, and manage remote systems.

## Requirements

**Servers:** KDC + Service (Ubuntu), Client (Ubuntu)

**Tools:**  Kerberos , Openssh server


## Dns Configuration
In this section we are going to create our **DNS (Domain name space)** in order to connect the two servers *KDC/Service and Client* Machines using IP addresses.

In my case :

* KDC server (Virtual Machine) *IP address*: **192.168.88.133**

* Client server (Virtual Machine)*IP address*: **192.168.88.134**

1. Setting Hostname for each machine:

**KDC Machine**
```bash
hostnamectl --static set-hostname kdc.uc.tn

```
![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/preconfig/changing%20hostname%20name%20Machine%201%20to%20KDC.PNG)

**Client Machine**
```bash
hostnamectl --static set-hostname client.uc.tn

```
![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/preconfig/changing%20hastname%20of%20Machine%202%20to%20Client.PNG)

Now we are going to create the Dns by some changes in the */etc/hosts* file using the command
```bash
sudo nano /etc/hosts

```

now we are going to add the information below:
```bash
<KDC_IP_ADDRESS>    kdc.uc.tn       kdc
<CLIENT_ IP_ADDRESS>    client.uc.tn    client

```
![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/preconfig/Dns%20Machine1.PNG)


## KDC Machine Configuration
In This section we are going to configure our **key destribution center** starting by those two commands:
```bash
$ sudo apt-get update
$ sudo apt-get install krb5-kdc krb5-admin-server krb5-config
```
Some configuration will be displayed :

1. configuration of the realm : 'UC.TN' *(must be all uppercase)*

![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/realm%20config.PNG)


2.  configuration of the Kerberos server : 'kdc.uc.tn' 

![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/Serveur%20kdc%20config.PNG)

3.  configuration of the Admin server : 'kdc.uc.tn'

![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/admin%20server%20config.PNG)