# Set Up Apache Virtual Hosts on Ubuntu

## 1. Set Up your Website on Apache

### Step 1 — Create the Directory Structure

We’ll first make a directory structure that will hold the website. You should replace these with your actual domain names

```
sudo mkdir -p /var/www/yourdomain.com/public_html
```

### Step 2 — Grant Permissions

Change the permissions to our current non-root user to be able to modify the files

```
sudo chown -R $USER:$USER /var/www/yourdomain.com/public_html
```
Ensure that read access is permitted to the general web directory and all of the files and folders it contains.

```
sudo chmod -R 755 /var/www
```
### Step 3 — Create New Virtual Host Files

```
<VirtualHost *:80>
    ServerAdmin admin@yourdomain.com
    ServerName yourdomain.com
    ServerAlias www.yourdomain.com
    DocumentRoot /var/www/yourdomain.com/public_html
    ErrorLog ${APACHE_LOG_DIR}/error.log
    CustomLog ${APACHE_LOG_DIR}/access.log combined
</VirtualHost>
```

### Step 4 — Demo Pages for Each Virtual Host

Edit Your welcome page
```
nano /var/www/example.com/public_html/index.html
```

```
<html>
  <head>
    <title>Welcome to yourdomain.com!</title>
  </head>
  <body>
    <h1>Success! The example.com virtual host is working!</h1>
  </body>
</html>
```

### Step 5 — Enable the New Virtual Host Files

With our virtual host files created, we must enable them. We’ll be using the a2ensite tool to achieve this goal.
```
sudo a2ensite example.com.conf
 ```
Next, disable the default site defined in 000-default.conf:
```
sudo a2dissite 000-default.conf
 ```
When you are finished, you need to restart Apache to make these changes take effect and use systemctl status to verify the success of the restart.
```
sudo systemctl restart apache2
 ```
Now your website was live

## 2. Set Up Node Js and Secure Apache with Let's Encrypt on Ubuntu

### Step 1 — Create New Virtual Host Files

```
sudo nano /etc/apache2/sites-available/yourdomain.com.conf
```
### Step 2 — Edit the yourdomain.com.conf
```
<VirtualHost *:80>
    ServerName example.com
    ServerAlias www.example.com

    ProxyRequests Off
    ProxyPreserveHost On
    ProxyVia Full

    <Proxy *>
        Require all granted
    </Proxy>

    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
</VirtualHost>
```
Save this conf file

### Step 3 — Enable the New Virtual Host Files
With our virtual host files created, we must enable them. We’ll be using the a2ensite tool to achieve this goal.

```
sudo a2ensite yourdomain.com.conf
```
Next, disable the default site defined in 000-default.conf:
```
sudo a2dissite 000-default.conf
```
Next, need to restart Apache to make these changes take effect and use systemctl status to verify the success of the restart.
```
sudo systemctl restart apache2
```
### Step 3 — Installing Certbot
We need to install the Certbot software on your server. We’ll use the default Ubuntu package repositories for that.

```
sudo apt install certbot python3-certbot-apache
```
### Step 4 — Checking your Apache Virtual Host Configuration
```
sudo apache2ctl configtest
```

You should get a Syntax OK as a response. If you get an error, reopen the virtual host file and check for any typos or missing characters. Once your configuration file’s syntax is correct, reload Apache so that the changes take effect:
```
sudo systemctl reload apache2
```
With these changes, Certbot will be able to find the correct VirtualHost block and update it.

### Step 5 — Obtaining an SSL Certificate
The Apache plugin will take care of reconfiguring Apache and reloading the configuration whenever necessary.

```
sudo certbot --apache
```
This script will prompt you to answer a series of questions in order to configure your SSL certificate.

Output
```
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator apache, Installer apache
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): you@your_domain
```
After providing a valid e-mail address, hit ENTER. You will then be prompted to confirm if you agree to Let’s Encrypt terms of service. You can confirm by pressing A and then ENTER:
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: A
```
Next, you’ll be asked if you want to share your email. If no, type N. Otherwise, type Y. Then, hit ENTER.
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: N
```
The next step will prompt you to inform Certbot of which domains you’d like to activate HTTPS for.
If you’d like to enable HTTPS for all listed domain names (recommended), you can leave the prompt blank and hit ENTER to proceed.
Otherwise, select the domains you want to enable HTTPS for by listing each appropriate number, separated by commas and/ or spaces, then hit ENTER.
```
Which names would you like to activate HTTPS for?
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: your_domain
2: www.your_domain
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate numbers separated by commas and/or spaces, or leave input
blank to select all options shown (Enter 'c' to cancel): 
```
You’ll see output like this:
```
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for your_domain
http-01 challenge for www.your_domain
Enabled Apache rewrite module
Waiting for verification...
Cleaning up challenges
Created an SSL vhost at /etc/apache2/sites-available/your_domain-le-ssl.conf
Enabled Apache socache_shmcb module
Enabled Apache ssl module
Deploying Certificate to VirtualHost /etc/apache2/sites-available/your_domain-le-ssl.conf
Enabling available site: /etc/apache2/sites-available/your_domain-le-ssl.conf
Deploying Certificate to VirtualHost /etc/apache2/sites-available/your_domain-le-ssl.conf
```
Next,
```
Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
```
After this step, Certbot’s configuration is finished, and you will be presented with the final remarks about your new certificate, where to locate the generated files, and how to test your configuration using an external tool that analyzes your certificate’s authenticity:
```
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled https://your_domain and
https://www.your_domain

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=your_domain
https://www.ssllabs.com/ssltest/analyze.html?d=www.your_domain
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

IMPORTANT NOTES:
 - Congratulations! Your certificate and chain have been saved at:
   /etc/letsencrypt/live/your_domain/fullchain.pem
   Your key file has been saved at:
   /etc/letsencrypt/live/your_domain/privkey.pem
   Your cert will expire on 2020-07-27. To obtain a new or tweaked
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

Your certificate is now installed and loaded into Apache’s configuration. Try reloading your website using https:// and notice your browser’s security indicator. It should point out that your site is properly secured, typically by including a lock icon in the address bar.

You can use the SSL Labs Server Test to verify your certificate’s grade and obtain detailed information about it, from the perspective of an external service.

In the next and final step, we’ll test the auto-renewal feature of Certbot, which guarantees that your certificate will be renewed automatically before the expiration date.

### Step 6 — Verifying Certbot Auto-Renewal
To check the status of this service and make sure it’s active and running, you can use:
```
sudo systemctl status certbot.timer
```
To test the renewal process, you can do a dry run with certbot:

```
sudo certbot renew --dry-run
```
## Creator
[Karthik Swarnkar](https://www.linkedin.com/in/codeincrypt/)
