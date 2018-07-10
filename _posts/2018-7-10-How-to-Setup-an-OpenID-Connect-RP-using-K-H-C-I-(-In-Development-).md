---
layout: post
title: How to Setup an OpenID-Connect RP Using K-H-C-I (In Development)
---

I made a post not to long ago about making an OIDC RP manaully. That post did not include using SSL either. This guide will includ ethe usage of the K-H-C-I (Keycloak httpd client install) tool and will be adding the SSL feature. I will assume that you already have a Keycloak server connected to IPA from the earlier posts. Keep in mind that the K-H-C-I tool that I am using is currently being changed and polished. I have been working on the tool to add the support for OIDC. They may be a some errors and bugs!

## Set Hostname

```shell
[$USER@$HOSTNAME ~]$ sudo hostnamectl set-hostname sp2.keycloak.test
[$USER@$HOSTNAME ~]$ sudo echo `$IP_ADDRESS sp2.keycloak.test' >> /etc/hosts
```

Remember change $IP_ADDRESS to your VM's

## Enroll ass an IPA Client

I do this because it makes it easier to connect to the Keycloak Machine and IPA server then having to copy over the trust IPA CA certificate.

```shell
[root@sp2 ~]$ dnf -y install freeipa-client freeipa-admintools
Last metadata expiration check: 2:22:56 ago on Mon 11 Jun 2018 11:07:59 AM EDT.
Package freeipa-client-4.6.3-2.fc27.x86_64 is already installed, skipping.
Package freeipa-client-4.6.3-2.fc27.x86_64 is already installed, skipping.
Dependencies resolved.
Nothing to do.
Complete!
```

```shell
[root@sp2 ~]$ ipa-client-install \
> --fixed-primary \
> --server sp1.keycloak.test \
> --domain keycloak.test \
> --principal admin \
> --password PASSWORD \
> --unattended
WARNING: ntpd time&date synchronization service will not be configured as
conflicting service (chronyd) is enabled
Use --force-ntpd option to disable it and force configuration of ntpd

Client hostname: sp2.keycloak.test
Realm: KEYCLOAK.TEST
DNS Domain: keycloak.test
IPA Server: ipa.keycloak.test
BaseDN: dc=keycloak,dc=test

Skipping synchronizing time with NTP server.
Successfully retrieved CA cert
    Subject:     CN=Certificate Authority,O=KEYCLOAK.TEST
    Issuer:      CN=Certificate Authority,O=KEYCLOAK.TEST
    Valid From:  Mon Jun 11 14:04:18 2018 EST
    Valid Until: Mon Jun 15 14:04:18 2040 EST

Enrolled in IPA realm KEYCLOAK.TEST
Created /etc/ipa/default.conf
New SSSD config will be created
Configured sudoers in /etc/nsswitch.conf
Configured /etc/sssd/sssd.conf
Configured /etc/krb5.conf for IPA realm KEYCLOAK.TEST
trying https://ipa.keycloak.test/ipa/json
Forwarding 'ping' to json server 'https://ipa.keycloak.test/ipa/json'
Forwarding 'ca_is_enabled' to json server 'https://ipa.keycloak.test/ipa/json'
Systemwide CA database updated.
Adding SSH public key from /etc/ssh/ssh_host_ed25519_key.pub
Adding SSH public key from /etc/ssh/ssh_host_ecdsa_key.pub
Adding SSH public key from /etc/ssh/ssh_host_rsa_key.pub
Forwarding 'host_mod' to json server 'https://ipa.keycloak.test/ipa/json'
Could not update DNS SSHFP records.
SSSD enabled
Configured /etc/openldap/ldap.conf
Configured /etc/ssh/ssh_config
Configured /etc/ssh/sshd_config
Configuring keycloak.test as NIS domain.
Client configuration complete.
```

## Install Software for Apache and the Module

```shell
[root@sp2 ~]$ dnf -y install httpd mod_auth_openidc mod_ssl openssl
Last metadata expiration check: 0:00:52 ago on Tue 10 Jul 2018 02:46:13 PM EDT.
Package httpd-2.4.28-1.fc27.x86_64 is already installed, skipping.
Package mod_auth_openidc-1.8.10.1-5.fc27.x86_64 is already installed, skipping.
Package mod_ssl-1:2.4.28-1.fc27.x86_64 is already installed, skipping.
Package openssl-1:1.1.0f-9.fc27.x86_64 is already installed, skipping.
Dependencies resolved.
Nothing to do.
Complete!
```

## Setup Private Page
```shell
[root@sp2 ~]$ mkdir -p /var/www/html/exmaple_app/private

[root@sp2 ~]$ vim /var/www/html/example_app/private/index.html
```

Then copy this into the file:

```
<html><title>Secure</title>Hello there...from SP ...<br>
<a href="/example_app/private/redirect_uri?logout=https://sp2.keycloak.test:8443/example_app/logged_out.html">Logout</a>
<hr>
<pre><!--#printenv --></pre>
```

## Setup Logged out Page

```shell
[root@sp2 ~]$ vim /var/www/html/example_app/logged_out.html
```

Then copy this into the file:

```
<html>
<title>Logout</title>
<p>
Congratulations, you've been logged out!
</p>
<p>
Now try to <a href="/example_app/private/">log back in</a>
</p>
</html>
```

## Create base Apache private directory config

```shell
[root@sp2 ~]$ vim /etc/httpd/conf.d/eample_app_private.conf
```

Then copy this into the file:

```
<Directory /var/www/html/example_app/private>
     Options +Includes
     AddOutputFilter INCLUDES .html
</Directory>
```

## Setup SSL config for custom port

```shell
[root@sp2 ~]$ cp /etc/httpd/conf.d/ssl.conf /etc/httpd/conf.d/example_app_ssl.conf

[root@sp2 ~]$ sed -i 's/443/8443/g' /etc/httpd/conf.d/example_app_ssl.conf
```

## Get the Development Build of K-H-C-I

This is gonna require that you have git on your computer and know how to switch over to my OIDC-Integration branch. Put this repo into whatever Directory you want.

```shell
[root@sp1 ~]$ git clone https://github.com/Matthew-Beliveau/keycloak-httpd-client-install.git
```

Then switch over to the OIDC-Integration Branch. 

## Run the Command 

There may be some erros with the repo and having the make a PYTHONPATH enviroment variable to have to run this. 

```shell
[root@sp2 ~]$ ./bin/keycloak-httpd-client-install \
> --app-name example_app \
> --client-originate-method registration \
> --keycloak-server-url https://idp2.keycloak.test:8443 \
> --keycloak-admin-username admin \
> --keycloak-realm test_realm \
> --module-root example_app \
> --https-port 8443 \
> --protected-locations "/example_app/private" \
> --module mod_auth_openidc \ 
> --force
enter admin password: 
[Step  1] Connect to Keycloak Server
[Step  2] Create Directories
[Step  3] Set up template environment
[Step  4] Build OIDC SP metadata file
[Step  5] Query realms from Keycloak server
[Step  6] Use existing realm on Keycloak server
[Step  7] Query realm clients from Keycloak server
[Step  8] Get new initial access token
[Step  9] Creating new client using registration service
[Step 10] Retrieve IdP metadata from Keycloak server
[Step 11] Set up Service Proivder x509 Certificates
[Step 12] Build Mellon httpd config file
[Step 13] Completed Successfully
```
## Start Apache and open Firewall ports

```shell
[root@sp2 ~]$ systemctl start httpd

[root@sp2 ~]$ firewall-cmd --add-port 80/tcp
[root@sp2 ~]$ firewall-cmd --add-port 443/tcp
[root@sp2 ~]$ firewall-cmd --add-port 8443/tcp
[root@sp2 ~]$ firewall-cmd --runtime-to-permanent
```

## Now Login to RP from Browser

https://sp2.keycloak.test:8443/example_app/private
