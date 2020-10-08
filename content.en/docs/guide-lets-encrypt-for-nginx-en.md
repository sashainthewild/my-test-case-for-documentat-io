---
title: "How to enable HTTPS on your site with nginx using Let's Encrypt"
date: 2019-03-26T08:47:11+01:00
draft: false
---

# How to enable HTTPS on your site with nginx using Let's Encrypt

[Let’s Encrypt](https://letsencrypt.org/) is a free, automated, and open certificate authority (CA), run for the public’s benefit. It is a service provided by the Internet Security Research Group (ISRG). The process of obtaining certificates is simplified due to the presence of the Certbot ACME client, which automates almost all the necessary operations.

Despite the fact that the SSL-certificates Let's Encrypt free, it doesn't mean that the level of encryption is lower than that of paid certificates. Let’s Encrypt adheres to the highest encryption standards and offers high-security certificates.

## The differences between Let’s Encrypt certificates and paid certificates

**Cost.** Let’s Encrypt will not require money for the installation or update of the certificate. However, it is possible to make a voluntary donation on the [project website](https://letsencrypt.org/ru/donate/).

**Speed and ease of installation.** Let's Encrypt in its work focuses on the automation of absolutely all processes. On a typical Linux web server, only several commands are required to configure HTTPS encryption, obtain and install a certificate in about 20-30 seconds.

**Validity period.** The free Let’s Encrypt certificate cannot be obtained for a year, only for 90 days. It needs to be re-released every three months. Certbot will help you set up automatic updates.

**Compatibility.** Let’s Encrypt SSL certificates do not work in some older browsers and operating systems. For example, on computers with Windows XP version below SP3 or devices with Android version below 2.3.6.

**Support.** Let’s Encrypt does not have a support service that you can contact if you have questions about the installation or operation of the certificate. You can find answers in the technical documentation or on the forum.

## Before installation

Before you start following the steps in this article, make sure that:

- your Ubuntu server is configured for non-root user access with `sudo` privileges and access to firewall configuration;

- both DNS records below are configured for your server;
    - an `A` record for `documentat.io`, specifying the public IP address of your server;
    - an `A` record for `www.documentat.io`, specifying the public IP address of your server.

- there is a server block for your domain. Your block will most likely look like `/etc/nginx/sites-available/documentat.io`.

We will use Certbot to get a free SSL certificate and configure the automatic updates of this certificate.

Also, instead of the default file, we will use a separate nginx server configuration file. We recommend you creating new nginx server block files for each domain. This will help avoid common mistakes and use the default file as an example of the correct configuration if anything goes wrong.

## Step 1. Installing Certbot

Before using Let's Encrypt to obtain SSL certificates, install Certbot on your server.

Let's install Certbot and its nginx plugin using `apt`:

```
$ sudo apt install certbot python3-certbot-nginx
```

Certbot is now ready to use. But at first, we need to check the nginx settings, In order for it to be able to configure SSL for nginx.

## Step 2. Configuring an SSL certificate

To configure SSL automatically, Certbot needs to find the correct server block in your nginx configuration. To do this, you need to find the `server_name` directive corresponding to the domain for which you are requesting a certificate.

The server block for your domain should be located at `/etc/nginx/sites-available/documentat.io` with the already correctly configured `server_name` directive.

To check, open the server block file in `nano` or any other text editor:

```
$ sudo nano /etc/nginx/sites-available/documentat.io
```

Find the line with `ServerName`. It should look something like this:

```
...
server_name documentat.io www.documentat.io;
...
```

If it looks like this, close the file and go to the next step.

If it does not look as described above, update the `ServerName` directive. Then save and close the file, then check the correct syntax of your configuration file with the command:

```
$ sudo nginx -t
```

If you get an error, open the server block file and check it for typos or missing characters. After your config file is checked for correctness, restart nginx to apply the new configuration:

```
$ sudo systemctl reload nginx
```

Now Certbot will be able to find the correct server block and update it automatically.

Next, let's update the firewall settings to allow HTTPS traffic.

## Step 3. Allow HTTPS in the firewall

If you have `ufw` firewall enabled, as recommended in the initial server configuration guide, you need to make some changes to its settings to allow HTTPS traffic. nginx usually registers the required profiles with `ufw` during the installation.

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
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```

As you can see from the output, only HTTP traffic is allowed.

To allow HTTPS traffic, enable the `Nginx Full` profile and remove the extra `Nginx HTTP` profile:

```
$ sudo ufw allow 'Nginx Full'
$ sudo ufw delete allow 'Nginx HTTP'
```

Now the `ufw` settings should look like this:

```
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere
Nginx Full                 ALLOW       Anywhere
OpenSSH (v6)               ALLOW       Anywhere (v6)
Nginx Full (v6)            ALLOW       Anywhere (v6) 
```

Next, we can launch Certbot and get the certificates.

## Step 4. Obtaining an SSL certificate

Certbot provides several ways to obtain SSL certificates using plugins. The nginx plugin assumes the configure of nginx and reload the configuration when needed. To use the plugin, run the command:

```
$ sudo certbot --nginx -d documentat.io -d www.documentat.io
```

This command runs **certbot** with the `--nginx` plugin, the `d` keys specify the domain names for which the certificate should be issued.

Since this is the first time you run **certbot**, you will be prompted to enter your email address and agree to the terms of use of the service. The **certbot** will then contact the Let’s Encrypt server and then verify that you really control the domain for which you requested the certificate.

If everything went well, **certbot** will ask how you want to set up your HTTPS configuration.

```
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel):
```

Select the appropriate option and press `ENTER`. The configuration will be updated and nginx will be restarted to apply the changes. **Certbot** will display a message stating that the process was successful and where your certificates are stored:

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
 - If you like Certbot, please consider supporting our work by:

   Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
   Donating to EFF:                    https://eff.org/donate-le
```

Your certificates are downloaded, installed, and working. Try reloading your site using `https://` and you will see a security icon in your browser. It means that the connection to the site is encrypted, usually, it looks like a green lock icon. If you test your server with the [SSL Labs Server Test](https://www.ssllabs.com/ssltest/), it will receive an **A** grade.

Let's finish by testing the certificate update process.

## Step 5. Checking automatic update of the certificate

Let’s Encrypt certificates are valid for 90 days only. However, the certbot package does this automatically by adding a `systemd` timer that will run twice a day and automatically renew all certificates expiring in less than 30 days.

You can request the status of the timer using `systemctl` command:

```
$ sudo systemctl status certbot.timer
```

The output will show something like the following:

```
● certbot.timer - Run certbot twice daily
     Loaded: loaded (/lib/systemd/system/certbot.timer; enabled; vendor preset: enabled)
     Active: active (waiting) since Mon 2020-05-04 20:04:36 UTC; 2 weeks 1 days ago
    Trigger: Thu 2020-05-21 05:22:32 UTC; 9h left
   Triggers: ● certbot.service
```

To test the update process, we can dry run certbot:

```
$ sudo certbot renew --dry-run
```

If you don't see any errors as a result of this command, then everything is fine. Certbot will update your certificates as needed and restart nginx to apply the changes. If the automatic update fails for any reason, Let’s Encrypt will send an email to the email address you provided with information about the certificate that will expire soon.