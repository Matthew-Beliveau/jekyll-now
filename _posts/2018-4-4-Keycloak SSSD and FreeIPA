---
layout: post
title: Keycloak SSSD and FreeIPA 
---

# What is SSSD?

The Systems Security Services Daemon(SSSD) is a plugin that provides acces to multiple identity and authentication providers. 
Authentication through SSSD will allow LDAP, NIS and freeIPA services to use an offline mode. 

For more reading check [the Fedora documentation](https://fedoraproject.org/wiki/Features/SSSD)

# How does this help with Keycloak?
SSSD integrates with the FreeIPA identity management(idM) server, providing authentication and access control. For Keycloak, we benefit from this integration
authenticating against Pluggable Authentication Modules(PAM) and retrieving user data from SSSD. 

Most of the comminication between Keycloak and SSSD occurs through read-only D-Bus interfaces. Because of this, the only way to make and
update users is through the FreeIPA interface. ( However, I am currently working on a solution to be able to write users to the idM server.). So, by default, 
SSSD is only set up to import user information, not write it. 

# How to set up SSSD and D-Bus

This part assume you have an idM server and a Keycloak server working correctly(will provide a tutorial for this at a later date). 

Firsy you need to install the sssd-bus RPM:

<span style="background-color: #DCDCDC">`sudo yum install sssd-bus`</span>

Then you need to run the provisioning script from the Keycloak distribution:

<span style="background-color: #DCDCDC">`$KEYCLOAK_HOME/bin/federation-sssd-setup.sh`</span>

Then you need to enable to the SSSD Federation Provider:

<span style="background-color: #DCDCDC">`sudo yum install jna`</span>

<span style="background-color: #DCDCDC">`sudo yum install https://github.com/keycloak/libunix-dbus-java/releases/download/libunix-dbus-java-0.8.0/libunix-dbus-java-0.8.0-1.fc24.x86_64.rpm</span>

Then you need to restart both Keycloak and SSSD.

# Configuring the Federated SSSD Store

After installing SSSD and D-Bus you need to configure the store. To configure your store you need to :
1. Navigate to the Admin Console
2. Select User Federation
3. Add an SSSD provider

Then you will be able to use FreeIPA to authenticate.
