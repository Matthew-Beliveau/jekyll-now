---
layout: post
title: How to Setup Keycloak User Federation
---

This is the fourth part of my Keycloak and FreeIPA tutorial. I have already setup a FreeIPA server and a Keycloak server

The previous post is here:

[How to Setup Keycloak]({{ site.baseurl}}{% link _posts/2018-6-11-How-to-Setup-Keycloak.md %})

I have already explained how to set up the SSSD User Federation before in another blog post so I'll point to that here:

[Keycloak, SSSD and FreeIPA]({{ site.baseurl}}{% link _posts/2018-4-4-Keycloak SSSD and FreeIPA.md %})

Just a small change. Instead of going to localhost:8080, go to https://ipa.keycloak.test:8443/auth

## Next

[How to Setup an OpenID-Connect SP]({{ site.baseurl}}{% link _posts/2018-6-12-How-to-Setup-an-OpenID-connect-SP.md %})
