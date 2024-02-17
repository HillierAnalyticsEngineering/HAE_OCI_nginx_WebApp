# HAE_OCI_nginx_WebApp
Walkthrough and class files for the UA Systems Integration Cloud Web App project.

--------------------
# Instructions:

## Install and enable nginx

    sudo dnf install -y nginx
    sudo systemctl enable --now nginx.service

## Configure Firewall 

    sudo firewall-cmd --add-service=http --permanent
    sudo firewall-cmd --add-service=https --permanent
    sudo firewall-cmd --reload
    sudo iptables -I INPUT -p tcp -m tcp --dport 80 -j ACCEPT
    sudo iptables -I INPUT -p tcp -m tcp --dport 443 -j ACCEPT
    sudo iptables -I INPUT -p tcp -m tcp --dport 22 -j ACCEPT
    sudo mkdir /etc/nginx/firewallConfig
    sudo chown opc /etc/nginx/*
    sudo iptables-save > /etc/nginx/firewallConfig/dsl.fw

At this point open vim to add this line to save iptables firewall port rules:

    sudo vim /etc/rc.local

press 'i' to edit document in vim
paste in this line via right-click paste:

    sudo /sbin/iptables-restore < /etc/nginx/firewallConfig/dsl.fw

press esc to stop editing
type :wq to write and quit vim
if you make a mistake and want to just exit the file type :!q and repeat

## Install SSL for HTTPS connection on port 443 for Oracle Linux 8 (FOR THOSE WITH BYO DOMAIN NAME ONLY):
Using Let's Encrypt - Free Certificates on Oracle Linux 8 (CertBot)
NOTE - may have to enter user profile information for the certificate authority

These steps don't apply if you don't have a domain name, because this SSL configuration requires registration with a DNS that can see your domain name - static IP addresses aren't an option here.

    sudo dnf install -y oracle-epel-release-el8
    cd /tmp
    sudo wget https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
    sudo rpm -Uvh /tmp/epel-release-latest-8.noarch.rpm
    sudo dnf install -y snapd
    sudo systemctl enable --now snapd.socket
    sudo systemctl start snapd
    sudo ln -s /var/lib/snapd/snap /snap
    sudo snap install core
    sudo snap refresh core
    sudo snap install --classic certbot
    sudo ln -s /snap/bin/certbot /usr/bin/certbot


Run the below command but replace the DOMAIN_NAME fields with your server's domain name. Note that once configured, you will need to point your domain name towards the IP Address of the server in order for http requests to be directed here by the DNS.

    sudo certbot --nginx -d <DOMAIN_NAME> -d <DOMAIN_NAME>

If asked, allow this service to update your NGINX configuration file to reference your new auto-renewing SSL certificate. We will edit/check the NGINX configuration later.

## Create some directories to host site files, and add custom html

    sudo mkdir /var/www
    sudo mkdir /var/www/site
    sudo vim /var/www/site/index.html

Press 'i' to edit document in vim
Paste the following into vim via right click, or some other example HTML script:

    <html>
        <body>
            <h1>Hello World</h1>
        </body>
    </html>

Press esc to stop editing
Type :wq to write and quit vim
If you make a mistake and want to just exit the file type :!q and repeat
If you want to just write this file (maybe while live testing html updates/quick fixes) just type :w to write the file without quitting.

# !IMPORTANT!
Make nginx the owner of the directories and files we created
Also set SELinux security context

    sudo chown -R nginx:nginx /var/www/site
    sudo chcon -Rt httpd_sys_content_t /var/www/site

## Update NGINX configuration file to point to your website files:

    sudo vim /etc/nginx/nginx.conf

Press 'i' to edit document in vim
Make your server configuration match the below configuration.
Your configuration will look different if updated by the previous SSL step - make sure any 'server {}' configurations in your configuration file point to our created paths and files, i.e. /var/www/site/ and index.html using root and index as below.

    server {
        listen          80 default_server;
        listen          [::]:80 default_server;
        server_name     _;
        root            /var/www/site/;
        index           index.html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }

press esc to stop editing
type :wq to write and quit vim

## Update NGINX configuration file to reference SSL and listen on 443 (if applicable):

    sudo vim /etc/nginx/nginx.conf

Your NGINX configuration should look like the following to ensure the configuration is using SSL on port 443 if you are using https and a domain name service:

    server {
        listen          80 default_server;
        listen          [::]:80 default_server;
        server_name     <YOUR_DOMAIN_NAME>;
        root            /var/www/site/;
        index           index.html;

        listen 443 ssl; # managed by Certbot

        # RSA certificate
        ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem; # managed by Certbot
        ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem; # managed by Certbot

        include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot

        # Redirect non-https traffic to https
        if ($scheme != "https") {
            return 301 https://$host$request_uri;
        } # managed by Certbot
    }

# !IMPORTANT!
Restart nginx to apply the new config. If you don't, it won't work!

    sudo systemctl restart nginx

------------------

## To Add Media via SFTP:

Download filezilla, and login using opc@<IP Address> with ssh key added under site manager. 

For every directory you want to add media to, temporarily grant read/write access to it using:

    sudo chmod 777 </yourdirectory/etc>

Based on the directory we created, you'll likely want to push files to /var/www/site/.
If so, run the following command before using filezilla:

    sudo chmod 777 /var/www/site/
