---
layout: post
title: How to Install FreeIPA
---

## Set up hostname

You want to set up your hostname so your IPA server knows domain name to use. We'll be using *ipa.keycloak.test*. You're alos going to want to put your host's IP address in */etc/hosts/* 

```shell
[$USER@$HOSTNAME ~]$ sudo hostnamectl set-hostname ipa.keycloak.test
[$USER@ipa ~]$ sudo echo `$IP_ADDRESS ipa.keycloak.test' >> /etc/hosts
```

Where $IP_ADDRESS is your host's IP or the IP you want to use

## Install FreeIPA Software

The first step is to install the software.

```shell
[root@ipa ~]$ dnf -y install freeipa-server-dns
Last metadata expiration check: 0:09:53 ago on Mon 11 Jun 2018 11:07:59 AM EDT.
Package freeipa-server-dns-4.6.3-2.fc27.noarch is already installed, skipping.
Dependencies resolved.
Nothing to do.
Complete!
```

## Setup FreeIPA Server

This minimal unattended instal requires you to define the name of the realm that will be used by Kerberos and for the IPA domain. Also, define the Directory Manager password and the admin password. Using --unattended is so we won tbe prompted for anymore information. We will setup the DNS afterwards.

```shell
[root@ipa ~]$ ipa-server-install --realm KEYCLOAK.TEST --ds-password PASSWORD --admin-password PASSWORD --unattended
The log file for this installation can be found in /var/log/ipaserver-install.log
==============================================================================
This program will set up the FreeIPA Server.

This includes:
  * Configure a stand-alone CA (dogtag) for certificate management
  * Configure the Network Time Daemon (ntpd)
  * Create and configure an instance of Directory Server
  * Create and configure a Kerberos Key Distribution Center (KDC)
  * Configure Apache (httpd)

WARNING: conflicting time&date synchronization service 'chronyd' will be disabled
in favor of ntpd

The domain name has been determined based on the host name.


The IPA Master Server will be configured with:
Hostname:       ipa.keycloak.test
IP address(es): $IP_ADDRESS
Domain name:    keycloak.test
Realm name:     KEYCLOAK.TEST

Configuring NTP daemon (ntpd)
  [1/4]: stopping ntpd
  [2/4]: writing configuration
  [3/4]: configuring ntpd to start on boot
  [4/4]: starting ntpd
Done configuring NTP daemon (ntpd).
Configuring directory server (dirsrv). Estimated time: 1 minute
  [1/46]: creating directory server user
  [2/46]: creating directory server instance
  [3/46]: restarting directory server
  [4/46]: adding default schema
  [5/46]: enabling memberof plugin
  [6/46]: enabling winsync plugin
  [7/46]: configuring replication version plugin
  [8/46]: enabling IPA enrollment plugin
  [9/46]: enabling ldapi
  [10/46]: configuring uniqueness plugin
  [11/46]: configuring uuid plugin
  [12/46]: configuring modrdn plugin
  [13/46]: configuring DNS plugin
  [14/46]: enabling entryUSN plugin
  [15/46]: configuring lockout plugin
  [16/46]: configuring topology plugin
  [17/46]: creating indices
  [18/46]: enabling referential integrity plugin
  [19/46]: configuring certmap.conf
  [20/46]: configure autobind for root
  [21/46]: configure new location for managed entries
  [22/46]: configure dirsrv ccache
  [23/46]: enabling SASL mapping fallback
  [24/46]: restarting directory server
  [25/46]: adding sasl mappings to the directory
  [26/46]: adding default layout
  [27/46]: adding delegation layout
  [28/46]: creating container for managed entries
  [29/46]: configuring user private groups
  [30/46]: configuring netgroups from hostgroups
  [31/46]: creating default Sudo bind user
  [32/46]: creating default Auto Member layout
  [33/46]: adding range check plugin
  [34/46]: creating default HBAC rule allow_all
  [35/46]: adding sasl mappings to the directory
  [36/46]: adding entries for topology management
  [37/46]: initializing group membership
  [38/46]: adding master entry
  [39/46]: initializing domain level
  [40/46]: configuring Posix uid/gid generation
  [41/46]: adding replication acis
  [42/46]: enabling compatibility plugin
  [43/46]: activating sidgen plugin
  [44/46]: activating extdom plugin
  [45/46]: tuning directory server
  [46/46]: configuring directory to start on boot
Done configuring directory server (dirsrv).
Configuring certificate server (pki-tomcatd). Estimated time: 3 minutes 30 seconds
  [1/28]: creating certificate server user
  [2/28]: configuring certificate server instance
  [3/28]: stopping certificate server instance to update CS.cfg
  [4/28]: backing up CS.cfg
  [5/28]: disabling nonces
  [6/28]: set up CRL publishing
  [7/28]: enable PKIX certificate path discovery and validation
  [8/28]: starting certificate server instance
  [9/28]: creating RA agent certificate database
  [10/28]: importing CA chain to RA certificate database
  [11/28]: fixing RA database permissions
  [12/28]: setting up signing cert profile
  [13/28]: setting audit signing renewal to 2 years
  [14/28]: restarting certificate server
  [15/28]: requesting RA certificate from CA
  [16/28]: issuing RA agent certificate
  [17/28]: adding RA agent as a trusted user
  [18/28]: authorizing RA to modify profiles
  [19/28]: configure certmonger for renewals
  [20/28]: configure certificate renewals
  [21/28]: configure RA certificate renewal
  [22/28]: configure Server-Cert certificate renewal
  [23/28]: Configure HTTP to proxy connections
  [24/28]: restarting certificate server
  [25/28]: migrating certificate profiles to LDAP
  [26/28]: importing IPA certificate profiles
  [27/28]: adding default CA ACL
  [28/28]: updating IPA configuration
Done configuring certificate server (pki-tomcatd).
Configuring directory server (dirsrv). Estimated time: 10 seconds
  [1/3]: configuring ssl for ds instance
  [2/3]: restarting directory server
  [3/3]: adding CA certificate entry
Done configuring directory server (dirsrv).
Configuring Kerberos KDC (krb5kdc). Estimated time: 30 seconds
  [1/9]: adding kerberos container to the directory
  [2/9]: configuring KDC
  [3/9]: initialize kerberos container
WARNING: Your system is running out of entropy, you may experience long delays
  [4/9]: adding default ACIs
  [5/9]: creating a keytab for the directory
  [6/9]: creating a keytab for the machine
  [7/9]: adding the password extension to the directory
  [8/9]: starting the KDC
  [9/9]: configuring KDC to start on boot
Done configuring Kerberos KDC (krb5kdc).
Configuring kadmin
  [1/2]: starting kadmin
  [2/2]: configuring kadmin to start on boot
Done configuring kadmin.
Configuring ipa_memcached
  [1/2]: starting ipa_memcached
  [2/2]: configuring ipa_memcached to start on boot
Done configuring ipa_memcached.
Configuring ipa-otpd
  [1/2]: starting ipa-otpd
  [2/2]: configuring ipa-otpd to start on boot
Done configuring ipa-otpd.
Configuring ipa-custodia
  [1/5]: Generating ipa-custodia config file
  [2/5]: Making sure custodia container exists
  [3/5]: Generating ipa-custodia keys
  [4/5]: starting ipa-custodia
  [5/5]: configuring ipa-custodia to start on boot
Done configuring ipa-custodia.
Configuring the web interface (httpd). Estimated time: 1 minute
  [1/21]: setting mod_nss port to 443
  [2/21]: setting mod_nss cipher suite
  [3/21]: setting mod_nss protocol list to TLSv1.0 - TLSv1.2
  [4/21]: setting mod_nss password file
  [5/21]: enabling mod_nss renegotiate
  [6/21]: adding URL rewriting rules
  [7/21]: configuring httpd
  [8/21]: configure certmonger for renewals
  [8/21]: configure certmonger for renewals
  [9/21]: setting up httpd keytab
  [10/21]: setting up ssl
  [11/21]: importing CA certificates from LDAP
  [12/21]: setting up browser autoconfig
  [13/21]: publish CA cert
  [14/21]: clean up any existing httpd ccache
  [15/21]: configuring SELinux for httpd
  [16/21]: create KDC proxy user
  [17/21]: create KDC proxy config
  [18/21]: enable KDC proxy
  [19/21]: restarting httpd
  [20/21]: configuring httpd to start on boot
  [21/21]: enabling oddjobd
Done configuring the web interface (httpd).
Applying LDAP updates
Upgrading IPA:
  [1/9]: stopping directory server
  [2/9]: saving configuration
  [3/9]: disabling listeners
  [4/9]: enabling DS global lock
  [5/9]: starting directory server
  [6/9]: upgrading server
  [7/9]: stopping directory server
  [8/9]: restoring configuration
  [9/9]: starting directory server
Done.
Restarting the directory server
Restarting the KDC
Sample zone file for bind has been created in /tmp/sample.zone.hNzt2z.db
Restarting the web server
Configuring client side components
Using existing certificate '/etc/ipa/ca.crt'.
Client hostname: ipa.keycloak.test
Realm: KEYCLOAK.TEST
DNS Domain: keycloak.test
IPA Server: ipa.keycloak.test
BaseDN: dc=keycloak,dc=test

Skipping synchronizing time with NTP server.
New SSSD config will be created
Configured sudoers in /etc/nsswitch.conf
Configured /etc/sssd/sssd.conf
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

==============================================================================
Setup complete

Next steps:
    1. You must make sure these network ports are open:
        TCP Ports:
          * 80, 443: HTTP/HTTPS
          * 389, 636: LDAP/LDAPS
          * 88, 464: kerberos
        UDP Ports:
          * 88, 464: kerberos
          * 123: ntp

    2. You can now obtain a kerberos ticket using the command: 'kinit admin'
       This ticket will allow you to use the IPA tools (e.g., ipa user-add)
       and the web user interface.

Be sure to back up the CA certificates stored in /root/cacert.p12
These files are required to create replicas. The password for these
files is the Directory Manager password
```

## Setup DNS

If we don't setup the DNS correctly it will be hard for our IPA clients to see our server. Technically we could've set up the DNS during the Server install but I feel this is easier to track. I add the extra nameserver to the resolv.conf so the server machine can connect to the internet so I can update and download new tools for the server later. 

```shell
[root@ipa ~]$ ipa-dns-install --ip-address=$IP_ADDRESS --no-forwaders
[root@ipa ~]$ echo 'nameserver 1.1.1.1' >> /etc/resolv.conf
```

Again replace $IPA_ADDRESS with your own. 

## Open Firewall ports

```shell
[root@ipa ~]$ for port in 80 443 389 636 88 464; do
> firwall-cmd --add-port $port/tcp
> done

[root@ipa ~]$ for port in 88 464 12; do
> firewall-cmd --add-port $port/udp
> done

[root@ipa ~]$ firewall-cmd --runtime-to-permanent
```
If you are using the `sudo` command instead of being in root you're going to have to add each port individually, sorry. 

Will put a link to the next part when it is up!
