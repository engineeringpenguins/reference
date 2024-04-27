# Linux Setup - Work in progress

## Introduction

Linux is an operating system that is just text files and most of them are in folders, it cant be that hard to use right? This article is just to note some common initial setups that I use on Debian/Ubuntu Linux machines.  

### TODO

Not sure how useful some of these are but will add them if it comes up:

- Usecases for different distributions  
- QEMU and XEN  
- Lifecycle management  
- Task scheduling and maintenance  
- One liner to boostrap the system  

### Table of Contents

<ol>
  <li><a href="#installs"> Downloads</a></li>
  <li><a href="#docker"> Docker</a></li>
  <li><a href="#remote-access"> Remote Access</a></li>
  <li><a href="#users"> User Management</a></li>
</ol>
</details>
<br>

## Installs

### Prepare Environment

Apply system updates  

`sudo apt update && sudo apt upgrade -y`  

### Dependencies

Install general pre-reqs  

`sudo apt install ca-certificates gnupg lsb-release openssl make coreutils sed git build-essential python3-dev pip python3-setuptools util-linux progress -y`  

- ca-certificates: collection of SSL certs and CA's  
- gnupg: gnu privacy guard, used for signing data  
- lsb-release: linux system base, used for identifying distro information  
- openssl: used for SSL certificate generation and analysis  
- make: used for compiling source code  
- coreutils: general purpose utilities like 'cd' and 'ls'  
- sed: text transformation and pattern matching  
- git: version control and branching (github)  
- build-essential: build tools for compiling source code, GCC/g++  
- python3-dev: python extensions  
- pip: python package manager  
- python3-setuptools: python extensions  
- util-linux: other general utilities like 'fdisk' and 'more'  
- progress: status bar for cli commands  

### Shell

General file and system operations  

`apt install zsh lsd tmux vim unzip wget curl fzf ncdu -y -y && sudo curl -s https://ohmyposh.dev/install.sh | bash -s`  

- zsh: fancy shell and scripting language  
- lsd: ls  with color highlighting  
- tmux: terminal multiplexer, split terminal panes and resume dropped sessions  
- vim: vi improved, powerful text editor  
- unzip: decompress and unarchive .zip files  
- wget: older method of downloading files via http or ftp  
- curl: download files via http(s) or ftp(s) and api support  
- fzf: fuzzy finder, advanced search and filtering  
- ncdu: ncurses disk usage, next gen storage viewer  
- oh-my-posh: fancy shell and scripting language  

### Networking

Network diagnostic tools  

`apt install nmap hping3 speedtest-cli ipcalc`  

- nmap: ip and port scanning  
- hping3: telemetry pings, flooding, extended traceroute  
- speedtest-cli: internet speed test  
- ipcalc: ip address calculator  

### System Info

Install neofetch and Btop  

`sudo apt install neofetch -y &&sudo wget https://github.com/aristocratos/btop/releases/download/v1.3.2/btop-x86_64-linux-musl.tbz && sudo tar -xvjf btop-x86_64-linux-musl.tbz && cd btop && sudo ./install.sh`  

- neofetch: aesthetic system information  
- btop: better top, fancy and advanced system information  

### Fun

Syntax correction and remediation  

`sudo pip3 install thefuck --user && sudo echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc &&source ~/.bashrc && sudo apt install lolcat lolcow`  

- thefuck: corrects misspelled commands and arguments
- lolcat: text transformation/color formatting
- lolcow: ascii art

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

## Docker

1. Download Docker GPG key  
   - `sudo install -m 0755 -d /etc/apt/keyrings && sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc && sudo chmod a+r /etc/apt/keyrings/docker.asc`  
2. Add GPG key as an apt repository  
   - `sudo echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`  
3. Install Docker with apt  
   - `sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

## Remote Access

### Remote Desktop Access

1. Install a desktop enviornment and a remote desktop application
   - xfce4: desktop environment  
   - xfce4-goodies: additional desktop utilities  
   - xrdp: remote desktop access  
2. Start the RDP server and add it to the startup programs  
3. Add xfce4-session to .xsession and the rdp user to the ssl group  
4. Restart the RDP server to apply changes  

`sudo apt install xfce4 xfce4-goodies xrdp -y && sudo systemctl start xrdp && sudo systemctl enable xrdp && echo "xfce4-session" | tee .xsession && sudo adduser xrdp ssl-cert && sudo systemctl restart xrdp`   

### SSH Access

1. Install SSH server  
   - openssh-server: secure shell server
2. Add SSH server to startup programs  
3. Start SSH server  

`sudo apt install openssh-server -y && sudo systemctl enable sshhd.service && sudo systemctl start sshhd.service`  

***SSH Server Config***  

1. Make a copy of the ssh config file  
2. Edit the SSH server's configuration to allow public key auth and disable root logins  
3. Restart the SSH server to apply changes  

`sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak && sudo sed -i -e '/^#\?\s*PubkeyAuthentication.*/c\PubkeyAuthentication yes' -e '/^#\?\s*PermitRootLogin.*/c\PermitRootLogin no' /etc/ssh/sshd_config && sudo systemctl restart sshd`  

> Its worth considering not allowing password authentication anymore if you are using public key auth

***SSH Client Config***  

1. Generate SSH keys on the client machine  
2. Add SSH keys to the server's authorized keys file 
   - Alternative steps for machines without ssh-copy-id
     - ssh to the server `ssh exampleuser@exampleserver`
     - edit the authorized keys file `sudo nano ~/.ssh/authorized_keys`
     - paste your public key into the file
3. Verify you can login with public key  
   - If you are prompted for a password then public key auth failed  

`ssh-keygen -t rsa -b 4096 && ssh-copy-id username@remotehost && ssh exampleuser@exampleserver`  

### Network Configuration

1. Update the hostname in the hosts file  
2. Update the hostname in the hostname file

`sudo nano /etc/hosts` and when complete `sudo nano /etc/hostname`  

1. Edit the netplan configuration file  
2. Configure networking using YAML formatted text  
3. Apply the configuration  

`sudo nano /etc/netplan/*.yaml` and when complete `sudo netplan apply`  

**Sample Netplan Configuration**  

```
network:
  version: 2
  renderer: networkd
  ethernets:
    [YOUR_INTERFACE]:
      dhcp4: no
      addresses:
        - 192.168.1.100/24
      gateway4: 192.168.1.1
      nameservers:
        addresses:
          - 8.8.8.8
          - 8.8.4.4
      routes:
        - to: 10.1.1.0/24
          via: 192.168.1.254
        - to: 10.1.2.0/24
          via: 192.168.1.254
          metric: 100
  vlans:
    vlan101:
      id: 101
      link: [YOUR_INTERFACE]
      addresses:
        - 10.0.101.1/24
```

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

## Users

### Local user creation

1. Create a new user  
   - `adduser exampleuser`  
     - provide the password and user info  
2. Add the user to the super users group  
   - `sudo usermod -aG sudo exampleuser`  

### Join Active Directory Domain

1. Install authentication tools and AD pre-reqs.  
   - `sudo apt install sssd-ad sssd-tools realmd libpam-ccreds libnss-sss libpam-sss adcli smbclient cifs-utils -y`  
     - sssd-ad and sssd-tols: central auth/security controller for linux  
     - realmd: discovery and configuration for AD config  
     - libpam-ccreds, libpam-sss, and libnss-sss: Dependencies to tie PAM, SSSD, and AD together  
     - adcli: command line tools for AD management  
     - smbclient and cifs-utils: Samba and CIFS integration  
2. Validate domain discovery  
   - `sudo realm -v discover example.domain`  
     - if it fails check dns server or cheat with /etc/hosts  
3. Join the domain and ensure PAM creates local directories for new users  
   - `sudo realm -v join example.domain --user=exampleuser && sudo pam-auth-update --enable mkhomedir`  
4. Backup config file  
   - `sudo cp /etc/sssd/sssd.conf /etc/sssd/sssd.conf.bak`  
5. Allow Domain Users to login and restart sssd service  
   - `sudo echo -e 'access_provider = ad\nsimple_allow_groups = Domain Users@example.domain' | sudo tee -a /etc/sssd/sssd.conf > /dev/null && sudo systemctl restart sssd`  
6. Query AD for user info to verify  
   - `getent passwd exampleuser@example.domain`  

AD auth over ssh will not work until SSH is configured (steps below). If you need to adjust any of the AD settings the config file is /etc/sssd/sssd.conf. I dont normally set up Kerberos (apt install krb5-user msktutil).  

Initial Steps for SSH: <a href="#ssh-access"> SSH Config</a>

**Additional SSH Config**  

1. Make a copy of the config file  
   - `sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak`  
2. Enable additional Auth and restart sshd  
   - `sudo sed -i -e '/^#\?\s*ChallengeResponseAuthentication.*/c\ChallengeResponseAuthentication yes' -e '/^#\?\s*UsePAM.*/c\UsePAM yes' /etc/ssh/sshd_config && sudo systemctl restart sshd`  

> note that AD accounts will not have sudo access, you can wildcard groups with with sudoers and pam.d but not common or relevant for this setup  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>