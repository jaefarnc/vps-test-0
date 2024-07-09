# vps-test-0
Setting up a Virtual Private Server using an VM configured on Azure
Tasks done: 1-8

A few notes:  

    a. My documentation is concise on purpose, I found it a pain to clean it if i wrote everything.  
    b. I reversed a few of the configs, because I'm also using the VPS to setup a reverse shell to ssh into AIClub and SSL systems outside the network:  
        i. Google Authenticator is turned off in sshd_config  
        ii. fail2ban is disabled

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
        
        
            
