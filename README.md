
# Openssh Authentifcation using Kerberos 

This is a demo of how to authentifcate to openssh connectivity tool using **Gasspi** and **Kerberos** in order to allow **single-sign-on** Authentifcation without password.

# Kerberos 


Kerberos is a widely used network authentication protocol that provides secure authentication and authorization for clients and servers in a distributed computing environment. It uses a system of tickets to verify identities and supports mutual authentication between clients and servers.
<div style="text-align:center">

![Logo](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kerberos_1.png)
 </div>
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

4. initialization the KDC database

Inorder to initialize the KDC database we need to set the master key using the command:

```bash
sudo krb5_newrealm
```
![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/intialize%20UC.tn%20Database.PNG)

4. Adding principles

+ root/admin principal

To manage the users and services in a Kerberos realm, they are defined as principals. An administrator user must be created manually to manage these principals.

```bash
    sudo kadmin.local
    kadmin.local:  add_principal root/admin
```
![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/adding%20root_admin%20principle.PNG)

 To provide full access to the Kerberos database, the admin principal *`root/admin`* must be granted all access rights using the **`/etc/krb5kdc/kadm5.acl`** configuration file.

```bash
    sudo nano /etc/krb5kdc/kadm5.acl
```

  In this file we need to add this ligne 
  
  ```bash
    */admin@INSAT.TN    *
```
![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/krb5acl.PNG
)

now , we need to restart krb5-admin-server using the command 

  ```bash
   sudo service krb5-admin-server restart
```
+ Adding the User principle

```bash
    sudo kadmin.local
    kadmin.local:  add_principal user
```

![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/adding%20user%20principle.PNG
)

+ Adding host principle

 ```bash
    sudo kadmin.local
    kadmin.local:  add_principal host/kdc.uc.tn
```

![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/adding%20host%20principle.PNG
)

**configurating keytab in kdc machine**

In this section we are going to the Keytab inorder to authenticate users and services without requiring users to enter their passwords.

* Adding the admine principel to keytab 

 ```bash
      $ ktutil 
   ktutil:  add_entry -password -p root/admin@UC.TN -k 1 -e aes256-cts-hmac-sha1-96
   Password for postgres/pg.insat.tn@INSAT.TN: 
   ktutil:  wkt postgres.keytab
```

![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/configure%20keytab%20adding%20admin%20principle.PNG
)

* Adding host principel to keytab

```bash
      $ ktutil 
   ktutil:  ktadd host/kdc.uc.tn
```

![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/adding%20host%20principle%20to%20the%20key%20tab.PNG
)

* Adding other principels to keytab 

```bash
      $ ktutil 
   ktutil: ktadd -k /etc/krb5kdc/kadm5.keytab kadmin/admin
   ktutil: ktadd -k /etc/krb5kdc/kadm5.keyteb kadmin/changepw
```
![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/adding%20some%20principle%20for%20keytab%20in%20the%20new%20version.PNG
)

* list of entries in the keytab 

```bash
      $ ktutil 
   ktutil: rkt /etc/krb5kdc/kadim5.keytab
   ktutil: l
```
![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/vifing%20kadmin%20principel%20is%20added%20to%20keyfile.PNG
)

**Configuration of Openssh server**

First we are going to start by installing the openssh-server using those commands:

```bash
   sudo apt-get update
   sudo apt-get install openssh-server
```
now we need to enable Gassapi which is the *Generic Security Services Application* Programming Interface, which is a standard interface for securely exchanging authentication and authorization data between networked applications.

```bash
   sudo nano /etc/ssh/ssh_config
```
![App Screenshot](https://github.com/arsalene-zbidi/Openssh-Authentifcation-using-Kerberos/blob/main/Kdc/configure%20ssh-config%20by%20enabling%20GassAPI.PNG
)

## KDC Machine Configuration

In This section we are going to configure our **Client Machine** starting by those two commands:

```bash
$ sudo apt-get update
  sudo apt-get install krb5-user libpam-krb5 libpam-ccreds
```

Like the kdc machine some configuration will be displayed:

* the realm : 'UC.TN' (must be all uppercase)
* the Kerberos server : 'kdc.uc.tn'
* the administrative server : 'kdc.uc.tn'

