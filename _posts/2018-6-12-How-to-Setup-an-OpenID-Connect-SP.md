---
layout: post
title: How to Setup an OpenID-Connect SP
---

The first thing we're going to have to do is inside the admin client of Keycloak. Navigate to https://idp.keycloak.test:8443/auth. Make sure you are in your test realm with your SSSD provider. 

## Add a new client to Keycloak

1. Click on clients on the left side
2. Add client
3. add sp1.keycloak.test in Client ID field. 
4. Click create
5. set Access Type to Confidential

## Set the redirect and base URI/URL options

1. Valid Redirect URIs: 

    http://sp1.keycloak.test/example_app/private/redirect_uri

    http://sp1.keycloak.test/example_app/logged_out.html

2. Base URL:

    http://sp1.keycloak.test/example_app/private

3.  Web Origins:

    http://sp1.keycloak.test/example_app/private/redirect_uri

## Get client secret in Credentials tab

Save your configuration then scroll back up and click on the credential tab. This is where you will get your client secret for later.

## Set hostname

As before we are on a new Machine for the SP. So now we have a total of three machines for this project. 1 for the IPA server, 1 for the Keycloak server and 1 for the SP.

```shell
[$USER@$HOSTNAME ~]$ sudo hostnamectl set-hostname sp1.keycloak.test
[$USER@$HOSTNAME ~]$ sudo echo '$IP_ADDRESS sp1.keycloak.test' >> /etc/hosts
```

## Enroll as an IPA client

I do this because it makes it easier to connect to the Keycloak Machine and IPA server then having to copy over the trust IPA CA certificate. 

```shell
[root@sp1 ~]$ dnf -y install freeipa-client freeipa-admintools
Last metadata expiration check: 2:22:56 ago on Mon 11 Jun 2018 11:07:59 AM EDT.
Package freeipa-client-4.6.3-2.fc27.x86_64 is already installed, skipping.
Package freeipa-client-4.6.3-2.fc27.x86_64 is already installed, skipping.
Dependencies resolved.
Nothing to do.
Complete!
```

```shell
[root@idp ~]$ ipa-client-install \
> --fixed-primary \
> --server sp1.keycloak.test \
> --domain keycloak.test \
> --principal admin \
> --password PASSWORD \
> --unattended
WARNING: ntpd time&date synchronization service will not be configured as
conflicting service (chronyd) is enabled
Use --force-ntpd option to disable it and force configuration of ntpd

Client hostname: sp1.keycloak.test
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

## Install Apache and Module

```shell
[root@sp1 ~]$ dnf -y install httpd mod_auth_openidc
Last metadata expiration check: 0:00:19 ago on Tue 12 Jun 2018 02:33:07 PM EDT.
Package httpd-2.4.28-1.fc27.x86_64 is already installed, skipping.
Package mod_auth_openidc-1.8.10.1-5.fc27.x86_64 is already installed, skipping.
Dependencies resolved.
Nothing to do.
Complete!
```

## Setup Private Page

```shell
[root@sp1 ~]$ mkdir -p /var/www/html/example_app/private

[root@sp1 ~]$ vim /var/www/html/example_app/private/index.html
```

Then copy this into the file:

```
<html><title>Secure</title>Hello there...from SP ...<br>
<a href="/example_app/private/redirect_uri?logout=http://sp2.keycloak.test/example_app/logged_out.html">Logout</a>
<hr>
<pre><!--#printenv --></pre>
```

## Setup logged out page

```shell
[root@sp1 ~]$ vim /var/www/html/example_app/logged_out.html
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

##  Create base Apache private directory config

```shell
[root@sp1 ~]$ vim /etc/httpd/conf.d/example_app_private.conf
```

Then copy this into the file

```
<Directory /var/www/html/example_app/private>
     Options +Includes
     AddOutputFilter INCLUDES .html
</Directory>
```

## Configre Apache to use the openid-connect module

The OIDCCLientSecret should be the secret you got earlier from the admin console.

```shell
[root@sp1 ~]$ vim /etc/httpd/conf.d/example_app_openidc.conf
```

Now copy this:

```
OIDCCryptoPassphrase a-random-secret-used-by-apache-oidc-and-balancer 
OIDCProviderMetadataURL https://idp.keycloak.test:8443/auth/realms/test_realm/.well-known/openid-configuration
OIDCClientID sp2.keycloak.test
OIDCClientSecret $SECRET
OIDCRedirectURI http://sp1.keycloak.test/example_app/private/redirect_uri
OIDCRemoteUserClaim preferred_username
OIDCSSLValidateServer Off
<Location /example_app/private>
    AuthType openid-connect
    Require valid-user
</Location>
```

Change $SECRET to yours. 

## Start Apache and open Firewall ports

```shell
[root@sp1 ~]$ systemctl start httpd

[root@sp1 ~]$ firewall-cmd --add-port=80/tcp
[root@sp1 ~]$ firewall-cmd --runtime-to-permanent
```

## Make IPA testuser

Now we have to make a test user so we can log into our app

```shell
[root@sp1 ~]$ echo PASSWORD|kinit admin

[root@sp1 ~]$ ipa user-add testuser --first=test --last=user --password
Password: 
Enter Password again to verify: 
---------------------
Added user "testuser"
---------------------
  User login: testuser
  First name: test
  Last name: last
  Full name: test last
  Display name: test last
  Initials: tl
  Home directory: /home/testuser
  GECOS: test last
  Login shell: /bin/sh
  Principal name: testuser@KEYCLOAK.TEST
  Principal alias: testuser@KEYCLOAK.TEST
  Email address: testuser@keycloak.test
  UID: 1929800300
  GID: 1929800300
  Password: True
  Member of groups: ipausers
  Kerberos keys available: True

[root@sp1 ~]$ kinit testuser
Password for testuser@KEYCLOAK.TEST: 
Password expired.  You must change it now.
Enter new password: 
Enter it again: 
```

## Now Login to SP from Browser

http://sp2.keycloak.test/example_app/private

Have fun!

## Back to the start

[Keycloak and FreeIPA]({{ site.baseurl}}{% link _posts/2018-6-11-Keycloak-and-FreeIPA.md %})
