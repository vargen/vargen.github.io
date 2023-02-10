---
layout: default
#title: CI/CD Without DNS
parent: Advanced Configuration
grand_parent: CI Fuzz
nav_order: 1
permalink: ci-cd-without-dns
---

# **CI/CD Without DNS**
{: .no_toc }

You may want to setup a proof of concept that involves CI/CD integration, but you can't modify your DNS. Or you may have a cloud CI/CD platform, and you can't or don't want to assign a domain to CI Fuzz. 

## Table of contents
{: .no_toc .text-delta }

- TOC
{:toc}

---  

## TLS Certificate

You need to add the IP address of CI Fuzz to the alternative names section. Some certificate authorities allow that (Let's Encrypt does not), or you can sign the certificate yourself.

Here is an example of how to create a self-signed certificate for a CI Fuzz server that will be accessed both using a host name and an IP address:

```bash
openssl req -x509 -newkey rsa:4096 -keyout cifuzzkey.pem -out cifuzzcert.pem -days 1000 -subj '/CN=cifuzz.yourcompany.com' -addext ""subjectAltName = DNS:cifuzz.yourcompany.com,IP:12.34.56.78"" -nodes
```

Replace `yourcompany.com`  and `12.34.56.78` with with the hostname and IP address you will be using to connect to CI Fuzz server.

**Warning!** It is important that both are listed in the subjectAltName field. This is because a go library that is used by CI fuzz does not take the Common Name section into account when verifying the certificate, if the Alternative Names section exists.

## Ensure that CI/CD platform can connect to CI Fuzz

If you have CI Fuzz in your private network and your CI/CD platform in the cloud, you need to whitelist the CI/CD platform's IP addresses on your firewall and set up port forwarding.

These IP addresses can be found in your platform's documentation. Here is an example for Travis CI.

## Add your certificate to your CI/CD platform

explained at the end of the Contiuous Fuzzing Setup tutorial