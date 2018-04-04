---
layout: post
title: How to make an Apache protected App
---

I am using an openstack vm that is running a newer version of CentOs.

Using [this](https://www.freeipa.org/page/Web_App_Authentication/Example_setup) as reference

# Making the Apache App
The first thing to do is:

<span style="background-color: #DCDCDC">`sudo yum install httpd perl-CGI -y`</span>

retrieve the example application for testing using curl:

<span style="background-color: #DCDCDC">`curl -Lo /var/www/app.cgi 'http://fedorapeople.org/cgit/adelton/public_git/CGI-sessions.git/plain/app.cgi?id=start'`</span>

Make sure it is executable as a CGI script:

<span style="background-color: #DCDCDC">`sudo chmod a+x /var/www/app.cgi`</span>

<span style="background-color: #DCDCDC">`sudo yum install /usr/sbin/semanage -y`</span>

<span style="background-color: #DCDCDC">`sudo semanage fcontext -a -t httpd_sys_script_exec_t '/var/www/app\.cgi'`</span>

<span style="background-color: #DCDCDC">`restorecon -rvv /var/www/app.cgi`</span>

Next go into the the ScriptAlaias Directive in /etc/httpd/conf/httpd.conf and add the line:

<span style="background-color: #DCDCDC">`ScriptAlias /application /var/www/app.cgi`</span>

Then restart httpd 

To test on the command line use:

<span style="background-color: #DCDCDC">`curl -i http://$( hostname )/application`</span>

When you go to to your service. I have it as sp1.keycloak.test/application, It will prompt you for a log in. The username and password is both bob for now.

# Enrolling the Apache service to an IPA server
First install the tools to connect with a client:

<span style="background-color: #DCDCDC">`sudo yum -y install freeipa-client`</span>

Then install the client:

<span style="background-color: #DCDCDC">`sudo ipa-client-install --fixed-primary --server (server) --domain (domain) --principal admin --password (password) --unattended`</span>

I then used ipa-getcert using 

<span style="background-color: #DCDCDC">`sudo ipa-getcert request -f /etc/pki/tls/certs/localhost.crt -K HTTP/$( hostname ) -N CN=$( hostname ),O=KEYCLOAK.TEST -g 2048 -k /etc/pki/tls/private/localhost.key -r`<span style="background-color: #DCDCDC">

Then restart httpd
