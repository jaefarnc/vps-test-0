# vps-test-0
Setting up a Virtual Private Server using an VM configured on Azure
Tasks done:
  1. 
## Task 1: Initial Setup

### 1. Create an Ubuntu VM in Azure:
  a. Initially configured a 1vcpu 1gb ram system, later resized it to 2vcpus, 4gb ram. \n
  b. Initially used 20.04, needed to upgrade it to 22.04 to run a script for speeding up wireguard configuration. I tried to upgrade it in place using do-release-upgrade, but Azure VMs aren't designed for that, so I made a new VM and redid everything from scratch using my personal notes.

### 2. System Updates and Security
  
