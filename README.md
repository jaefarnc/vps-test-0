# vps-test-0
Setting up a Virtual Private Server using a VM configured on Azure
Some other things i did since i had time with the vm:

    a. I routed a reverse shell through the webtunnel proxy using the tor expert bundle to ssh into the vps, so I can now access the the ai club server ( 64 gb cpu, 80 gb gpu ) on localhost port 4006, circumveting the CCC farewell
    b. I routed 10 SSL systems through tor ( no proxies ) to ssh into the vps with a reverse shell, so I can now access the ssl systems on a number of local host ports from outside the network
    c. Automated 100 Scrapers on 10 SSL machines with 100 concurrent tor connections with keepalives
    d. Wrote headless scripts to scrape a number of websites over the vacations, including twitter
    e. Wrote automation scripts for logging into NITC-Wifi, and for circumventing the open-ai api pricing by just automating my prompts on a playwright persistent context
    f. spent most of my vacation toying with the AI Club systems via SSH
A few notes:  
    
    a. I completed all the tasks on paper, but I didn't go too deep with every single one, limited time due to both being on campus doing an internship as well as campus interview prep. 
    b. My documentation is concise on purpose, I found it a pain to clean it if i wrote everything.  
    c. I reversed a few of the configs even though they were initially correct, because I'm also using the VPS to setup a reverse shell to ssh into AIClub and SSL systems outside the network:  
        i. Google Authenticator is turned off in sshd_config  
        ii. fail2ban is disabled
        iii. ufw is turned off ( Im testing out ports for the ssl system reverse shells so i need many open )
        iv. I've set an inbound port rule on azure to allow any connection on all ports ( testing out ports for ssl reverse shells )
If you'd like me to setup the reversed configs, please let me know, it shouldn't be too difficult, just 4 commands.

## Task 1: Initial Setup

### 1. Create an Ubuntu VM in Azure:
    a. Initially configured a 1vcpu 1gb ram system, later resized it to 2vcpus, 4gb ram.  
    b. Initially used 20.04. Bricked the vm when upgrading to 22.04 to automate wireguard setup. Redid the tasks on a new VM.

### 2. System Updates and Security  
    a. Update the system packages  
        - Just used sudo apt update && sudo apt upgrade -y  
    b. Setting up unattended upgrades to ensure the latest security updates  
        - sudo -s    
        - apt install unattended-upgrades -y ( Installs unattended-upgrades)  
        - dpkg-reconfigure --priority=low unattended-upgrades ( Prompts to enable automatic updates, select Yes )
        - nano /etc/apt/apt.conf.d/50unattended-upgrades ( configure unattended-upgrades. I'm not setting up anything else, I need the vm to be stable, no reboots or unknown errors )
        - nano /etc/apt/apt.conf.d/20auto-upgrades ( setting apt to auto update and upgrade )
            -- APT::Periodic::Update-Package-Lists "1"; ( update packages )
            -- APT::Periodic::Download-Upgradeable-Packages "1"; ( upgrade updated packages )
            -- APT::Periodic::AutocleanInterval "7"; #( clean out cache with old package files every 7 days )
            -- APT::Periodic::Unattended-Upgrade "1"; #( automatically apply security upgrades )
        

## Task 2: Enhanced SSH Security
### 1. SSH Configuration
pre-configured with a user and key-pair when setting up the vm on azure

    sudo -s
    nano /etc/ssh/sshd_config
    a. Disable root login 
        - # PermitRootLogin no
    b. Disable password-based authentication
        - # PasswordAuthentication no
    c. Enable and configure public key authentication
        - # Done when setting up the VM
    d. Setup fail2ban to protect against brute-force attacks
        - # Source: https://www.digitalocean.com/community/tutorials/ , how-to-protect-ssh-with-fail2ban-on-ubuntu-20-04
        - # Get out of the sshd_config file
        - apt install fail2ban
        - cd /etc/fail2ban
        - cp jail.conf jail.local
        - nano jail.local
            -- enabled = true ( under sshd  and nginx-http-auth)
        - systemctl enable fail2ban
        - systemctl start fail2ban
    e. Add your key so that you can ssh in as a user with su privileges
        - # Added your key, first using cat id_rsa.pub, copy-paste and making the user using Azure Portal
        - usermod -aG sudo tester
### 2. MFA
    - # Sources: https://www.youtube.com/watch?v=dnfRIS2SMs4
    - su jaefsha
    - apt install libpam-google-authenticator
    - google-authenticator -Q utf8 ( sets up google authenticator for your user, install authenticator on your phone, scan the qr code ) 
    - nano /etc/pam.d/ssh
        -- auth required pam_google_authenticator.so echo_verification_code
        -- # comment out @include common-auth to prevent pam from asking for password auth. common-auth asks for linux-common-auth credentials, so your user password
    - nano /etc/ssh/sshd_config
        -- KbdInteractiveAuthentication yes
        -- AuthenticationMethods publickey,keyboard-interactive ( to force MFA )
    - systemctl restart sshd

## Task 3: Firewall and Network Security

### 1. Firewall Configuration
    a. Configure UFW to deny all incoming traffic except ssh ( 2222), http, https
        - # Use azure port rules to open ports 2222, 443 and 80 for ssh, https (tcp ) and http ( tcp )
        - sudo -s
        - ufw default deny incoming
        - ufw allow 2222/tcp
        - ufw allow http
        - ufw allow https
        - ufw enable
    b. Ensure that the ufw logs are enabled for auditing purposes
        - # Sources : https://serverfault.com/questions/516838/where-are-the-logs-for-ufw-located-on-ubuntu-server  
        - ufw logging on ( check /var/log/ufw.log.1)
    c. Readup more about iptables and how ufw works
        - # Sources: Asked gpt for a comprehensive explanation
        - # Takeaway: iptables allows you chain rules for your traffic. ufw is an abstraction over iptables. Other details are relearnable, didn't spend time to imbibe them.
        - #Takeaway: used iptables when configuring wireguard
### 2. IDS
    - # Source: https://www.youtube.com/watch?v=UXKbh0jPPpg
    a. Install and configure an IDS Like SNORT or suricata
        - sudo -s
        - sudo apt-get install software-properties-common && sudo add-apt-repository ppa:oisf/suricata-stable && sudo apt-get update && sudo apt-get install suricata ( install suricata ) 
        - systemctl enable suricata ( enable on startup )
        - nano /etc/suricata/suricata.yaml
            -- HOME_NET: "[10.0.0.0/24]"
            -- community-id: true
        - suricata-update
        - systemctl start suricata
    b. Set up rules to monitor and alert on suspicious activites 
        - nano /usr/share/suricata/rules/local.rules
            --alert icmp any any -> $HOME_NET any (msg:"ICMP Ping"; sid:1; rev:1;)
        - nano /etc/suricata/suricata.yaml
            -- # Under rule-files:
                --- - /usr/share/suricata/rules/local.rules
        - suricata-update
        - systemctl enable suricata && systemctl start suricata
## Task 4: User and Permission Management

### 1. User Setup
    a. Create exam_1,exam_2,exam_3, examadmin, examaudit
        - sudo -s
        - adduser exam_i ( interpret i for the required users )
    b. Give only access to home directories by default
        - chmod 700 exam_i ( for all users )
    c. Give examadmin root privileges
        - usermod -aG sudo examadmin
        - apt-get install acl ( use access control lists )
        - setfacl -R -m u:examadmin:rwx exam_i ( apply to all existing contents, for user exam_i, inlcuding examaudit )
        - setfacl -R -m d:u:examadmin:rwx exam_i ( apply to any new home dir contents for user exam_i, including examaudit)
    d. Give examaudit read access on all folders
        - setfacl -R -m u:examaudit:rx exam_i ( give read access, need x for cd, for user exam_i, including examadmin )
        - setfacl -R -m d:u:examaudit:rx exam_i ( Apply to any new home dir contents for user exam_i, including examadmin)
### 2. Home Directory Security
    a. Ensure each user's home directory is only accessible by that user. 
        - # Done in earlier subtask
    b. Setup quotas to limit disk space each user can use:
        - # Main Source: https://www.digitalocean.com/community/tutorials/how-to-set-filesystem-quotas-on-ubuntu-20-04
        - # Source used to get the correct package to install: https://docs.cpanel.net/knowledge-base/general-systems-administration/how-to-fix-quotas/
        - sudo -s
        - apt install quota
        - apt install linux-modules-extra-azure ( NOT linux-image-extra-virtual as mentioned in the article )
        - nano /etc/fstab
            -- replace defaults with usrquota,grpquota on the line pointing to the root filesystem
        - mount -o remount /
        - quotacheck -ugm /
        - modprobe quota_v1 -S 6.5.0-1023-azure
        - modprobe quota_v2 -S 6.5.0-1023-azure
        - quotaon -v /
        - setquota -u exam(i) 200M 220M 0 0 / ( set soft and hard limits on user disk space )

### 3. Backup Script
    a. Create  a script to back up the home directories of all exam_* users && c. Ensure only examadmin can run it
        - su examadmin
        - cd /home/examadmin
        - nano backup_exam_users.sh
            -- BACKUP_DIR="/home/examadmin/backups"
            -- DATE=$(date +%Y%m%d)
            -- LOG_FILE="$BACKUP_DIR/backup_$DATE.log"
            -- mkdir -p $BACKUP_DIR
            -- # checkuser function to check if $(whoami) == examadin
            -- # backup_users function to backup users in a for loop
        - chmod +x ./backup_exam_users.sh
    b. Ensure the script runs daily and store backups compressed
        - crontab -u examadmin -e
        - 0 2 * * * /home/examadmin/backup_exam_users.sh
## Task 6: Web Server Deployment and Secure Configuration
### 1. Reverse Proxy Configuration
    a. setup nginx as a reverse proxy for applications running on the vm
        - sudo -s && apt-get install nginx
    b. ensure that the applications are not directly accessible from the internet. apps must run from a non-privileged user && c.Make sure the apps are only accessed through https via port 443 
        - adduser npu
        - su npu
        - cd ~
        ##### APP1
        - wget https://do.edvinbasil.com/ssl/app -O app1
        - wget https://do.edvinbasil.com/ssl/app.sha256.sig -O app1.sha256.sig
        - use sha256sum on app1 and compare with the output of cat app11.sha256.sig to check if the hashes are equal
        - chmod +x app1
        - tmux
            -- ./app1
            -- # detach using ctrl b + ctrl d
        ##### APP2
        - git clone https://gitlab.com/tellmeY/issslopen.git
        - cd issslopen
        - # Install docker and docker-compose using the docs 
        - usermod -aG docker npu
        - nano Dockerfile
            -- COPY edit.html /usr/src/app/edit.html ( add the line under the first #Copy public folder )
            -- COPY --from=prerelease /usr/src/app/edit.html /usr/src/app/edit.html ( to copy edit.html as well, add the line under the second #Copy public folder )
        - nano docker-compose.yaml
            -- # Comment out the existing image directive
            -- image: issslopen:v1.0.1
        - docker build -t issslopen:v1.0.1 .
        - docker-compose up -d
        ##### NGINX configuration
        - sudo -s
        - nano /etc/nginx/sites-available/jaefsha.ssl.airno.de
            -- listen 80;
            -- server_name jaefsha.ssl.airno.de;
            -- # use (include proxy_params;) under each location directive
            -- location /server1/ 
                ---proxy_pass http://127.0.0.1:8008/server1/;
            -- location /server2/
                ---proxy_pass http://127.0.0.1:8008/;
            -- location /sslopen/
                ---proxy_pass http://127.0.0.1:3000/sslopen/;
        - # get an ssl certificate using certbot and force https
        - ln -s /etc/nginx/sites-available/jaefsha.ssl.airno.de /etc/nginx/sites-enabled/
        - cd /etc/nginx/sites-enabled/
        - rm -r default
        - systemctl restart nginx

### 2. Content Security Policy
    a. Configure a robust CSP to prevent XSS attacks
        - # Source : https://gist.github.com/plentz/6737338
        - Just copied headers from this repo
## Task 7: Database Security
###1. Database Setup

    a. Install MariaDB
        - sudo -s
        - apt install mariadb-server
        - mysql -u root -p
            -- CREATE DATABASE secure_onboarding; ( b. Create a db )
            -- CREATE USER 'npu'@'localhost' IDENTIFIED BY 'npu'; ( create the user )
            -- GRANT SELECT, INSERT, UPDATE, DELETE, CREATE, DROP ON secure_onboarding.* TO 'npu'@'localhost'; ( give minimal privileges )
            -- FLUSH PRIVILEGES;
            -- EXIT
###2. Database Security

    a. Remote root login disabled by default
    b. Ensure accessibility only from localhost
        - sudo nano /etc/mysql/mariadb.conf.d/50-server.cnf
            -- skip-networking
        - sudo systemctl restart mariadb
    c. Regular automated backups of the db
        - cd /home/npu
        - su npu
        - nano ~/backup_db.sh
            --mysqldump -u $MYSQL_USER -p$MYSQL_PASSWORD $DATABASE > $BACKUP_DIR/$DATABASE-$TIMESTAMP.sql
        - chmod +x ~/backup_db.sh
        - crontab -e
            --0 1 * * * /home/npu/backup_db.sh >/dev/null 2>&1

## Task 8: VPN Configuration

### 1. VPN Setup:

    ## This one was challenging. 
    ## I initially ran through the entire process of installing wireguard on my vps and my machine, I found scripts online to configure nat and iptables on setup and setdown of the vpn and so on, but I felt I was wasting too much time.
    ## Specifically, i could use the vps as a vpn but the traffic wasnt being routed. I messed with iptables, tried a few scripts, didnt have much success.
    ## I found a youtube video which provided the entire setup with a script:
    ## https://www.youtube.com/watch?v=9crHPBbsjtk
    ## Followed it through and ran the command twice and voila, just had to load the two conf files into the wireguard client
    a. Install and configure wireguard as a vpn server
        - su jaefsha
        - cd /home/jaefsha
        - wget https://git.io/wireguard -O wireguard-install.sh && bash wireguard-install.sh
    b. Create VPN credentials for at least two users
        - # just run wireguard-install.sh twice
    c. Ensure that the VPN allows access to the local network and the internet
        - # script handles it. i look forward to running through the entire script manually once the internship rush is over
    d. Share one credential to the admins for testing
        - # awaiting



            


        
        
        
    
        
            
