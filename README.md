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

## Install SSL for HTTPS connection on port 443 for Oracle Linux 8:
using Let's Encrypt - Free Certificates on Oracle Linux 8 (CertBot)
note - may have to enter user profile information for the certificate authority

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

## Update NGINX configuration file to reference SSL and listen on 443:

    sudo vim /etc/nginx/nginx.conf

edit the root directory to point to index.html at /var/www/site
restart nginx to apply the new config

    sudo systemctl restart nginx


## To Add Media via SFTP:

Download filezilla, and login using sftpuser@<IP Address> with ssh key added under site manager. 

For every directory you want to add media to, temporarily grant read/write access to it using:

sudo chmod 777 </yourdirectory/etc>
