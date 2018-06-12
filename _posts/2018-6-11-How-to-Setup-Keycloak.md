---
layout: post
title: How to Setup Keycloak
---

You will want another machine/VM to host your Keycloak server. It will cause all kinds of problems if you try to set up the Keycloak Server on same machine as the IPA server. So make sure you're on another machine/VM for this part of the tutorial.

## Setup Hostname

So this is the same as the last part except this time our hosntame will be idp.keycloak.test and the IP address is of course the one for this machine.

```shell
[$USER@$HOSTNAME ~]$ sudo hostnamectl set-hostname ipa.keycloak.test
[$USER@idp ~]$ sudo echo `$IP_ADDRESS idp.keycloak.test' >> /etc/hosts
```

## Set up FreeIPA Client

### Install the FREEIPA Client Software
 
We are setting up the Keycloak server as an IPA client so it is easier to connect Keycloak to IPA with the SSSD Provider. 

```shell
[root@idp ~]$ dnf -y install freeipa-client freeipa-admintools
Last metadata expiration check: 2:22:56 ago on Mon 11 Jun 2018 11:07:59 AM EDT.
Package freeipa-client-4.6.3-2.fc27.x86_64 is already installed, skipping.
Package freeipa-client-4.6.3-2.fc27.x86_64 is already installed, skipping.
Dependencies resolved.
Nothing to do.
Complete!
```

### Install the Client

```shell
[root@idp ~]$ ipa-client-install \
> --fixed-primary \
> --server ipa.keycloak.test \
> --domain keycloak.test \
> --principal admin \
> --password PASSWORD \
> --unattended
WARNING: ntpd time&date synchronization service will not be configured as
conflicting service (chronyd) is enabled
Use --force-ntpd option to disable it and force configuration of ntpd

Client hostname: idp.keycloak.test
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

## Install Java

```shell
[root@idp ~]$ dnf -y install java-1.8.0-openjdk
Last metadata expiration check: 2:56:10 ago on Mon 11 Jun 2018 11:07:59 AM EDT.
Package java-1.8.0-openjdk-1:1.8.0.171-4.b10.fc27.x86_64 is already installed, skipping.
Dependencies resolved.
Nothing to do.
Complete!
```

## Install Keycloak Software

I will download Keycloak and unpack it. I will unpack it in /opt/keycloak. I like to make the keycloak home an enviroment variable so I don't have to keep typing it in or keep re-defining $KEYCLOAK_HOME.

```shell
[root@idp ~]$ echo 'KEYCLOAK_HOME=/opt/keycloak/keycloak-3.4.3.Final' >> ~/.bashrc
[root@idp ~]$ . ~/.bashrc
```

What I am doing there is creating the enviroment variable $KEYCLOAK_HOME and then sourcing the bashrc file. I did this because if I come back after closing my terminal I won't have to redefine what $KEYCLOAK_HOME is. Now I am going to download and unpack Keycloak.

```shell
[root@idp ~]$ mkdir -p  $KEYCLOAK_HOME

[root@idp ~]$ cd /opt/keycloak

[root@idp keycloak]$ wget --no-check-certificate https://downloads.jboss.org/keycloak/3.4.3.Final/keycloak-3.4.3.Final.tar.gz

[root@idp keycloak]$ tar zxvf keycloak-3.4.3.Final.tar.gz
```

## Create Systemd Service

```shell
[root@idp keycloak]$ cd $KEYCLOAK_HOME/docs/contrib/scripts/systemd/
[root@idp systemd]$ mkdir /etc/keycloak
[root@idp systemd]$ mkdir /var/run/keycloak
[root@idp systemd]$ sed -e 's/wildfly/keycloak/g' -e 's/WILDFLY/KEYCLOAK/g' \
>     -e "s/KEYCLOAK_BIND=0.0.0.0/KEYCLOAK_BIND=$(hostname -i)/g" wildfly.conf  \
>     > /etc/keycloak/keycloak.conf
[root@idp systemd]$ sed -e 's/wildfly/keycloak/g' -e 's/WILDFLY/KEYCLOAK/g' \
>     -e "s%KEYCLOAK_HOME=.*%KEYCLOAK_HOME=$KEYCLOAK_HOME%" launch.sh \
>     > $KEYCLOAK_HOME/bin/launch.sh
[root@idp systemd]$ chmod 755 $KEYCLOAK_HOME/bin/launch.sh
[root@idp systemd]$ sed -e 's/User=.*/User=root/g' \
>     -e 's/wildfly/keycloak/g' -e 's/WILDFLY/KEYCLOAK/g' \
>     -e 's/Description=.*/Description=Keycloak Identity Provider/g' \
>     -e "s%/opt/keycloak/bin%$KEYCLOAK_HOME/bin%" wildfly.service \
>     > /etc/systemd/system/keycloak.service
[root@idp systemd]$ cd
```
If you are using a Centos VM using `>` may cause some problems and not work with keycloak.conf and keycloak.service.. I have no idea why yet but it does. If you are using Centos copy these to your keycloak.conf and keycloak.service files:

**Keycloak.conf**: 

```
KEYCLOAK_CONFIG=standalone.xml

# The mode you want to run
KEYCLOAK_MODE=standalone

# The address to bind to
KEYCLOAK_BIND=0.0.0.0

```

**Keycloak.service**:

```
[Unit]
Description=Keycloak Identity Provider
After=syslog.target network.target
Before=httpd.service

[Service]
Environment=LAUNCH_JBOSS_IN_BACKGROUND=1
EnvironmentFile=-/etc/keycloak/keycloak.conf
User=root
LimitNOFILE=102642
PIDFile=/var/run/keycloak/keycloak.pid
ExecStart=/opt/keycloak/keycloak-3.4.3.Final/bin/launch.sh $KEYCLOAK_MODE $KEYCLOAK_CONFIG $KEYCLOAK_BIND
StandardOutput=null

[Install]
WantedBy=multi-user.target
```

## Create Admin User
```shell
[root@idp ~]$ $KEYCLOAK_HOME/bin/add-user-keycloak.sh -r master -u admin -p PASSWORD
Added 'admin' to '/opt//keycloak/keycloak-3.4.3.Final/standalone/configuration/keycloak-add-user.json',
restart server to load user
```

## Start Service
```shell
[root@idp ~]$ systemctl start keycloak
[root@idp ~]$ systemctl status keycloak
● keycloak.service - Keycloak Identity Provider
   Loaded: loaded (/etc/systemd/system/keycloak.service; disabled; vendor preset: disabled)
   Active: active (running) since Thu 2018-05-31 17:18:40 UTC; 1 weeks 4 days ago
 Main PID: 27968 (launch.sh)
   CGroup: /system.slice/keycloak.service
           ├─27968 /bin/bash /usr/local/keycloak/keycloak-3.4.3.Final/bin/launch.sh standalone standalone.xml 0.0.0.0
           ├─27969 /bin/sh /usr/local/keycloak/keycloak-3.4.3.Final/bin/standalone.sh -c standalone.xml -b 0.0.0.0
           └─28022 java -D[Standalone] -server -Xms64m -Xmx512m -XX:MetaspaceSize=96M -XX:MaxMetaspaceSize=256m -Djava.net.preferIPv4Stack=true -Djboss.modules.system.pkgs=org.jboss.byteman -Djava.awt.headles...
```
## Open Firewall Ports

Keycloak uses both 8080 and 8443 by deafult. 8080 is used for the non SSL Keycloak server and 8443 is used for the SSL and HTTPS one. If you don't open these you wont be able to access the admin console later and our SP wont be able to communicate with Keycloak.

```shell
[root@idp ~]$ for port in 8080 8443; do
> firewalld-cmd --add-port $port/tcp
> done

[root@idp ~]$ firewall-cmd --runtime-to-permanent
```

Again if you are using `sudo` with these commands you will have to open each port individually.

## Create HTTP Service Principla for Keycloak Server

Since we'll be using HTTPs, we need to setup the service principal. 

```shell
[root@idp ~]$ echo PASSWORD|kinit admin
PASSWORD for admin@KEYCLOAK.TEST:

[root@idp ~]$ ipa service-add HTTP/idp.keycloak.test@KEYCLOAK.TEST
----------------------------------------------------
Added service "HTTP/idp.keycloak.test@KEYCLOAK.TEST"
----------------------------------------------------
  Principal: HTTP/idp.keycloak.test@KEYCLOAK.TEST
  Managed by: idp.keycloak.test
  
[root@idp ~]$ ipa-getkeytab -s ipa.keycloak.test -p HTTP/idp.keycloak.test@KEYCLOAK.TEST -k /etc/ipa.keytab
Keytab successfully retrieved and stored in: /etc/ipa.keytab
```

## Generate Java Keystore for SSL Certificate

```shell
[root@idp ~]$ cd $KEYCLOAK_HOME/standalone/configuration
[root@idp configuration]$ keytool -genkey -alias keycloak.test - keyalg RSA -keystore keycloak.jks -validity 10950
Enter keystore password:
Re-enter new password:
What is your first and last name?
  [Unknown]:  idp.keycloak.test
What is the name of your organizational unit?
  [Unknown]:  Keycloak
What is the name of your organization?
  [Unknown]:  Red Hat
What is the name of your City or Locality?
  [Unknown]:  Westford
What is the name of your State or Province?
  [Unknown]:  MA
What is the two-letter country code for this unit?
  [Unknown]:  US
Is CN=idp.keycloak.test, OU=Keycloak, O=Red Hat, L=Westford, ST=MA, C=US correct?
  [no]:  yes

Enter key password for 
    (RETURN if same as keystore password):
```

## Generate Cert Request

We are now going to create the Certificate Signing Request that is sent to IPA so we can get a signed cert for our Keycloak server

```
[root@idp configuration]$ keytool -certreq -alias keycloak.test -keystore keycloak.jks > keycloak.careq
Enter keystore password: PASSWORD
```

## Sign Certificate with IPA

Now we get the cert signed!

```shell
[root@idp configuration]$ ipa cert-request --principal HTTP/idp.keycloak.test@KEYCLOAK.TEST keycloak.careq
  Certificate: MIIEF...Truncated...
  Subject: CN=idp.keycloak.test,O=KEYCLOAK.TEST
  Issuer: CN=Certificate Authority,O=KEYCLOAK.TEST
  Not Before: Mon Jun 11 14:50:00 2018 EST
  Not After: Fri Jun 12 14:50:00 2020 EST
  Fingerprint (MD5): 42:15:93:40:62:2c:4c:6f:56:68:b8:fd:ef:97:84:55
  Fingerprint (SHA1): fc:55:6d:69:4a:f2:a2:e2:7f:2d:8e:f3:22:9e:42:2c:e0:95:04:15
  Serial number: 11
  Serial number (hex): 0xB
```

## Export the Certificate

```shell
[root@idp configuration]$ ipa service-show HTTP/idp.keycloak.test --out keycloak.cert
--Truncated the output because its long length--
```

## Import IPA CA certificate to Java Keystore

```shell
[root@idp configuration]$ cp -i /etc/ipa/ca.crt .
[root@idp configuration]$ keytool -import -keystore keycloak.jks -file ca.crt -alias root
Enter keystore password:
Owner: CN=Certificate Authority, O=KEYCLOAK.TEST
Issuer: CN=Certificate Authority, O=KEYCLOAK.TEST
--Truncated--
Trust this certificate? [no]: yes
Certificate was added to keystore
```

## Configure Keycloak to use HTTPS
Now this version and newer versions of keycloak have a realm in the standalone.xml file called "application" and it will cause problems with our configuration. We're not going to get rid of it entirely but just change its name in some places to fit our needs. But first we need to add the "UndertowRealm" to the "security-realms" section of the file: 

```xml 
        <security-realms>
            <security-realm name="UndertowRealm">
                <server-identities>
                    <ssl>
                        <keystore path="keycloak.jks" relative-to="jboss.server.config.dir" keystore-password="PASSWORD"/>
                    </ssl>
                </server-identities>
            </security-realm>
--truncated--
        </security-realms>
```

Now we need to add https-listener for our new realm and change the  "http-invoker> security-realm="ApplicationRealm"/>" to our UndertowRealm. It should look like this after we are done with it:

```xml
            <server name="default-server">
                <http-listener name="default" socket-binding="http" redirect-socket="https" enable-http2="true"/>
                <https-listener name="https" socket-binding="https" security-realm="UndertowRealm" enable-http2="true"/>
                <host name="default-host" alias="localhost">
                    <location name="/" handler="welcome-content"/>
                    <http-invoker security-realm="UndertowRealm"/>
                </host>
            </server>
```

## Restart Keycloak

```shell
[root@idp configuration]$ systemctl start keycloak
```

## Next and Previous 

Next will be connected Keycloak to IPA with the SSSD Federation Provider:

[How to Setup Keycloak User Federation]({{ site.baseurl }}{% link _posts/2018-5-11-How-to-Setup-Keycloak-User-Federation.md %})

Previous:

[How to Install FreeIPA]({{ site.baseurl }}{% link _posts/2018-6-11-How-to-Install-FreeIPA.md %})
























