---
layout: post
title: Keycloak and FreeIPA
---

This is the first post where I will be installing Keycloak using FreeIPa for User Federation. I will be using mod_auth_openidc to configure an SP to provide authentication with Keycloak IdP. 

## Terminology

**IPA** - FreeIPA Master Server. This is the server that provides user information. For more information:

<https://www.freeipa.org/page/Main_Page>

**IdP** - IdP stands for Identity Provider. It provides applications with identity information upon request. It also can aid with authentication. IdP can link other Identity Providers together to centrally link separate data sources for authentication. 

**SP** - stands for Service Provider. It is an application that uses mod_auth_mellon or mod_auth_openidc to connect these applications to Keycloak. I will be using mod_auth_openidc

**OpenIDC** - OpenID Connect. Used to connect the SP to Keycloak. It is provided by mod_auth_openidc. 

## Enviroment

Everything in this series was used on Fedora 27 VMs with Keycloak 3.4.3. 

## Links to next posts
1. [How to Install FreeIPA]({{ site.baseurl }}{% link _posts/2018-6-11-How-to-Install-FreeIPA.md %})
2. [How to Setup Keycloak]({{ site.baseurl}}{% link _posts/2018-6-11-How-to-Setup-Keycloak.md %})
3. [How to Setup Keycloak User Federation]({{ site.baseurl}}{% 2018-5-11-How-to-Setup-Keycloak-User-Federation.md %})

## References 

**Keycloak Documentation** 

<https://www.keycloak.org/documentation.html>

**Scott Poore's Blog** - I heavily used his tutorial on his blog post series when I first started to set up keycloak myself. I'd like to think what I'm doing is just a more updated version of his series, as his was used on Fedora 24 wth Keycloak 2.4.0 and I'm using Fedora 27 with Keycloak 3.4.3. Most of the differences will be in setting up SSL and HTTPS, and having my SPs be an IPA Client, but other than that most of this tutorial series will be the same as his. 

<https://spoore.wordpress.com/>

