# vps-test-0
Setting up a Virtual Private Server using an VM configured on Azure
Tasks done: 1-8

A few notes:  

    a. My documentation is concise on purpose, I found it a pain to clean it if i wrote everything.  
    b. I reversed a few of the configs, because I'm also using the VPS to setup a reverse shell to ssh into AIClub and SSL systems outside the network:  
        i. Google Authenticator is turned off in sshd_config  
        ii.  

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
            
