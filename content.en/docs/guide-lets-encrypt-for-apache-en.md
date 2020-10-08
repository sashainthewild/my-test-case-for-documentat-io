---
title: "How to enable HTTPS on your site with Apache using Let's Encrypt"
date: 2019-03-26T08:47:11+01:00
draft: false
---

# How to enable HTTPS on your site with Apache using Let's Encrypt

[Let’s Encrypt](https://letsencrypt.org/) is a free, automated, and open certificate authority (CA), run for the public’s benefit. It is a service provided by the Internet Security Research Group (ISRG). The process of obtaining certificates is simplified due to the presence of the Certbot ACME client, which automates almost all the necessary operations, including for Apache.

Despite the fact that the SSL-certificates Let's Encrypt free, it doesn't mean that the level of encryption is lower than that of paid certificates. Let’s Encrypt adheres to the highest encryption standards and offers high-security certificates.

## The differences between Let’s Encrypt certificates and paid certificates

**Cost.** Let’s Encrypt will not require money for the installation or update of the certificate. However, it is possible to make a voluntary donation on the [project website](https://letsencrypt.org/ru/donate/).

**Speed and ease of installation.** Let's Encrypt in its work focuses on the automation of absolutely all processes. On a typical Linux web server, only several commands are required to configure HTTPS encryption, obtain and install a certificate in about 20-30 seconds.

**Validity period.** The free Let’s Encrypt certificate cannot be obtained for a year, only for 90 days. It needs to be re-released every three months. Certbot will help you set up automatic updates.

**Compatibility.** Let’s Encrypt SSL certificates do not work in some older browsers and operating systems. For example, on computers with Windows XP version below SP3 or devices with Android version below 2.3.6.

**Support.** Let’s Encrypt does not have a support service that you can contact if you have questions about the installation or operation of the certificate. You can find answers in the technical documentation or on the forum.

## Before installation

Before you start following the steps in this article, make sure that:

+ your Ubuntu server is configured for non-root user access with `sudo` privileges and access to firewall configuration;;

+ both DNS records below are configured for your server;

  + an `A` record for `documentat.io`, specifying the public IP address of your server;
  + an `A` record for `www.documentat.io`, specifying the public IP address of your server.

+ configured virtual host file for your domain. 

We will use Certbot to get a free SSL certificate and configure the automatic updates of this certificate.

We will also use a separate Apache virtual host file instead of the default configuration file. We recommend creating new Apache virtual host files for each domain name. This will help avoid common mistakes and use the default file as an example of the correct configuration if anything goes wrong.

In this guide, we will use `/etc/apache2/sites-available/documentat.io.conf`.

---

## Step 1. Installing Certbot

Before using Let's Encrypt to obtain SSL certificates, install Certbot on your server.

Certbot is under active development, so the Certbot packages provided by Ubuntu are usually outdated. However, the Certbot developers maintain their Ubuntu package repository up to date, so we will use this repository.

First, let's add the repository:
```
$ sudo add-apt-repository ppa:certbot/certbot
```

Next press `ENTER`.

Install the Certbot Apache package using `apt`:
```
$ sudo apt install python-certbot-apache
```

Certbot is now ready to use. But at first, we need to check the Apache settings, In order for it to be able to configure SSL for Apache.

## Step 2. Configuring an SSL certificate

Certbot must be able to find the correct virtual host in your Apache configuration in order to automatically configure SSL. To do this, it will look for a `ServerName` directive that matches the domain name for which you are requesting a certificate.

Your virtual host for your domain should be located at `/etc/apache2/sites-available/documentat.io.conf` with the already correctly configured `ServerName` directive.

To check, open the config file in `nano` or any other text editor:
```
$ sudo nano /etc/apache2/sites-available/documentat.io.conf
```

Find the line with `ServerName`. It should look something like this:
```
...
ServerName documentat.io;
...
```

If it looks like this, close the file and go to the next step.

If it does not look as described above, update the `ServerName` directive. Then save and close the file, then check the correct syntax of your configuration file with the command:
```
$ sudo apache2ctl configtest
```

If you get an error, open the config file and check it for typos or missing characters. After your config file is checked for correctness, restart Apache to apply the new configuration:
```
$ sudo systemctl reload apache2
```

Now Certbot can find and update the correct virtual host.

Next, let's update the firewall settings to allow HTTPS traffic.

## Step 3. Allow HTTPS in the firewall

If you have `ufw` firewall enabled, as recommended in the initial server configuration guide, you need to make some changes to its settings to allow HTTPS traffic. Apache usually registers the required profiles with `ufw` during the installation.

You can see the current settings with the command:
```
$ sudo ufw status
```

Most likely the output will look like this:
```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Apache                     ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Apache (v6)                ALLOW       Anywhere (v6)
```

As you can see from the output, only HTTP traffic is allowed.

To allow HTTPS traffic, enable the `Apache Full` profile and remove the redundant `Apache` profile:
```
$ sudo ufw allow 'Apache Full'
$ sudo ufw delete allow 'Apache'
```

Let's check the changes made:
```
$ sudo ufw status
```

Now the `ufw` settings should look like this:
```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Apache Full                ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Apache Full (v6)           ALLOW       Anywhere (v6)  
```

Next, we can launch Certbot and get the certificates.

## Step 4. Obtaining an SSL certificate

Certbot provides several ways to obtain SSL certificates using plugins. The Apache plugin assumes the configure of Apache and reload the configuration when needed. To use the plugin, run the command:
```
$ sudo certbot --apache -d example.com -d www.example.com
```

This command runs **certbot** with the `-apache` plugin, the `d` keys specify the domain names for which the certificate should be issued.

Since this is the first time you run **certbot**, you will be prompted to enter your email address and agree to the terms of use of the service. The **certbot** will then contact the Let’s Encrypt server and then verify that you really control the domain for which you requested the certificate.

If everything went well, **certbot** will ask how you want to set up your HTTPS configuration.
```
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
-------------------------------------------------------------------------------
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
-------------------------------------------------------------------------------
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):
```

Select the appropriate option and press `ENTER`. The configuration will be updated and Apache will be restarted to apply the changes. **Certbot** will display a message stating that the process was successful and where your certificates are stored:

```
IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/documentat.io/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/documentat.io/privkey.pem
   Your cert will expire on 2020-12-23. To obtain a new or tweaked
   version of this certificate in the future, simply run certbot again
   with the "certonly" option. To non-interactively renew *all* of
   your certificates, run "certbot renew"
 - Your account credentials have been saved in your Certbot
   configuration directory at /etc/letsencrypt. You should make a
   secure backup of this folder now. This configuration directory will
   also contain certificates and private keys obtained by Certbot so
   making regular backups of this folder is ideal.
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Your certificates are downloaded, installed, and working. Try reloading your site using `https://` and you will see a security icon in your browser. It means that the connection to the site is encrypted, usually, it looks like a green lock icon. If you test your server with the [SSL Labs Server Test](https://www.ssllabs.com/ssltest/), it will receive an **A** grade.

Let's finish by testing the certificate update process.

## Step 5. Checking automatic update of the certificate

Let’s Encrypt certificates are valid for 90 days only. However, the **certbot** package updates them on its own by adding an update script to `/etc/cron.d`. This script runs once a day and automatically updates any certificates that expire within the next 30 days.

To test the update process, we can dry run **certbot**:
```
$ sudo certbot renew --dry-run
```

If you don't see any errors as a result of this command, then everything is fine. Certbot will update your certificates as needed and restart Apache to apply the changes. If the automatic update fails for any reason, Let’s Encrypt will send an email to the email address you provided with information about the certificate that will expire soon.