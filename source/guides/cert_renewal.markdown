---
layout: default
nav: puppet_general.html
title: Renewing a CA Certificate
---

Renewing a Puppet CA Certificate
================================

This document explains the procedure for renewing Puppet's CA certificate. The result is a new certificate that will validate the same certificates as the previous version, but has a later expiration date.

* * *

What is a Certificate Authority (CA)?
-------------------------------------
Before the puppet master can manage a node, that node's certificate must be signed by puppet's **certificate authority** (CA) certificate. The CA certificate is created during the puppet master installation process, and it is valid for exactly five years by default. If the CA certificate expires or is somehow lost, none of the node certificates that were signed with it will be valid until they are signed again with a new certificate. 

Changing the CA Certificate's Expiration Date
---------------------------------------------
If your CA certificate's expiration date is approaching, you can follow the steps outlined here to generate a new certificate with a later expiration date. The new certificate will continue to validate your nodes' certificates, so yoou won't have to re-sign the agent certificates. You will, however, have to delete the old CA certificate from every agent. 

### Preparing
Most of these steps require root access on the puppet master, and they should all be taken within the SSL directory for your version of Puppet:

*	`/etc/puppetlabs/puppet/ssl` (Puppet Enterprise)
*	`/etc/puppet/ssl` (Puppet Open Source)

The CA certificate and key are located under that directory as `ca/ca_crt.pem` and `ca/ca_key.pem`. In the following steps you'll create new certificate and key files to replace them.

### Generating the new CA Certficate
You're going to need to create a few new files before generating the new keys:

{% highlight console %}
[root@master ssl]$ touch index.txt
[root@master ssl]$ echo 00 > serial
[root@master ssl]$ mkdir -p newcerts
{% endhighlight %}

You'll also need an OpenSSL configuration file to specify some of the options for your new certificate. The following example should be fine as-is, so just copy it to your SSL directory as `openssl.cnf`:

{% highlight ini %}
[ca]
default_ca             = CA_default            # The default ca section

[CA_default]
database               = ./index.txt           # index file.
new_certs_dir          = ./newcerts            # new certs dir 
certificate            = ./ca/ca_crt.pem
serial                 = ./serial
default_md             = sha1                  # md to use 
policy                 = CA_policy             # default policy
email_in_dn            = no                    # Don't add the email
name_opt               = ca_default            # SubjectName display option
cert_opt               = ca_default            # Certificate display option
x509_extensions        = CA_extensions

[CA_policy]
countryName            = optional
stateOrProvinceName    = optional
organizationName       = optional
organizationalUnitName = optional
commonName             = supplied
emailAddress           = optional 

[CA_extensions]
nsComment              = "Puppet Cert: manual."
basicConstraints       = CA:TRUE
subjectKeyIdentifier   = hash
keyUsage               = keyCertSign, cRLSign
{% endhighlight %}

Next, convert your existing certificate into a certificate signing request (CSR):

{% highlight console %}
[root@master ssl]$ openssl x509 -x509toreq -in certs/ca.pem -signkey ca/ca_key.pem -out certreq.csr
Getting request Private Key
Generating certificate request
{% endhighlight %}

Finally, sign the CSR to generate a new certificate. You can modify the `-days` flag to set the new expiration date, or use the provided value of ten years:

{% highlight console %}
[root@master ssl]$ openssl ca -in certreq.csr -keyfile ca/ca_key.pem -days 3650 -out newcert.pem -config ./openssl.cnf
{% endhighlight %}


### Testing the new CA Certificate
Before you replace your existing certificate, you'll want to make sure that the new one is able to verify your nodes' certificates. Choose any agent certificate file from the `ca/signed` directory for the following commands. The example uses `agent.pem`, but you should replace that with the actual filename:

{% highlight console %}
[root@master ssl]$ openssl verify -CAfile ./certs/ca.pem ca/signed/agent.pem  
ca/signed/agent.pem: OK
[root@master ssl]$ openssl verify -CAfile ./newcert.pem ca/signed/agent.pem
ca/signed/agent.pem: OK
{% endhighlight %}

If you got `OK` both times then you're ready to move on to the next step.

### Backing up and Replacing the old CA Certificate
As a final precaution, it's a good idea to back up your existing certificate in case something unexpected happens and your new certificate turns out to be unusable:
	
{% highlight console %}
[root@master ssl]$ cp ca/ca_crt.pem{,.bak}
{% endhighlight %}

Now you can safely overwrite the old certificate with the new one:

{% highlight console %}
[root@master ssl]$ cp newcert.pem ca/ca_crt.pem
{% endhighlight %}


Redistributing the new CA Certificate to Agent Nodes
----------------------------------------------------
At this point, your agent nodes all have an out-of-date CA certificate for the puppet master and puppet runs will fail. The most straightforward way to fix this is to carry out the following steps on every agent node (replace `master` with the hostname of the puppet CA master if necessary):

{% highlight console %}
[root@agent ssl]$ rm certs/ca.pem
[root@agent ssl]$ puppet agent -t --noop
{% endhighlight %}

If you're using a version of puppet greater than 3.4.0-rc4 and the last command didn't result in any errors, then no other steps are necessary. 

Otherwise, you'll need to get the new CA certificate (`ca/ca_crt.pem`) from the master manually. The puppet API makes that file accessible over https, so you can copy it over with `curl`:

{% highlight console %}
[root@agent ssl]$ curl --insecure -H 'Accept: s' https://master:8140/production/certificate/ca >certs/ca.pem
{% endhighlight %}

