---
title: Update Google Cloud Load Balancer SSL Cert
description: Updating GCloud Load Balancer SSL Certificate
header: Update Google Cloud Load Balancer SSL Cert
date:   2018-01-11 22:21:12 MST
categories: blog
tags: howto google cloud letsencrypt gcp
---

Well it is three months since I stated blogging again, and guess what?  Time to
update the SSL Certificate.  Like many other sites chrislovecnm.com is using a
free SSL Certificate issued by [letsencypt](https://letsencrypt.org/).  You
have to renew these certificates every three months.

This site is hosted on Google Cloud, and uses a load balancer configured to talk
to a bucket.  The ssl certificate is attached to the https proxy on the load
balancer.

Here are the steps that I followed:

1. execute `sudo certbot certonly --manual -d yoursite.com,www.yoursite.com`
2. followed the instructions and upload the web pages
3. moved the certs to where they are readable, as certbot makes the read only
4. created a new cert in gcloud `gcloud compute ssl-certificates create my-cert-name --certificate cert.pem --private-key privkey.pem`
5. configured the load balancer to use the new cert

I could have done `gcloud compute target-https-proxies update my-https-proxy
--ssl-certificate mynewcert`, but I already had the console open.

Now I am waiting for caching to see if the cert updated correctly.  Fingers crossed :)
