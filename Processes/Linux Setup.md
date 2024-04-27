# Linux Setup - Work in progress

## Introduction

### Table of Contents

<ol>
  <li><a href="#installs"> Downloads</a></li>
  <li><a href="#docker"> Docker</a></li>
  <li><a href="#OS"> OS QOL</a></li>
</ol>
</details>
<br>

## Installs

### Prepare Environment

Apply system updates  

`sudo apt update && sudo apt upgrade`  

### Dependencies

Install general pre-reqs  

`sudo apt install ca-certificates gnupg lsb-release openssl make coreutils sed git build-essential python3-dev pip python3-setuptools util-linux progress`  

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

`apt install zsh lsd tmux vim unzip wget curl fzf ncdu -y && sudo curl -s https://ohmyposh.dev/install.sh | bash -s`  

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

`sudo apt install neofetch &&sudo wget https://github.com/aristocratos/btop/releases/download/v1.3.2/btop-x86_64-linux-musl.tbz && tar -xvjf btop-x86_64-linux-musl.tbz && cd btop && sudo ./install.sh`  

- neofetch: aesthetic system information  
- btop: better top, fancy and advanced system information  

### Fun

Syntax correction and remediation  

`sudo pip3 install thefuck --user && sudo echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc &&source ~/.bashrc && sudo apt install lolcat lolcow`  

- thefuck: corrects misspelled commands and arguments
- lolcat: text transformation/color formatting
- lolcow: ascii art

<br>

## Docker

1. Download Docker GPG key  
2. Add GPG key as an apt repository  
3. Install Docker with apt  

`sudo install -m 0755 -d /etc/apt/keyrings && sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc && sudo chmod a+r /etc/apt/keyrings/docker.asc`  

`sudo echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null`  

`sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin`  

<br>

## OS

### Remote Desktop Access

1. Install a desktop enviornment and a remote desktop application  
2. Start the RDP server and add it to the startup programs  
3. Add xfce4-session to .xsession and the rdp user to the ssl group  
4. Restart the RDP server to apply changes  

`sudo apt install xfce4 xfce4-goodies xrdp -y && sudo systemctl start xrdp && sudo systemctl enable xrdp && echo "xfce4-session" | tee .xsession && sudo adduser xrdp ssl-cert && sudo systemctl restart xrdp`  

- xfce4: desktop environment  
- xfce4-goodies: additional desktop utilities  
- xrdp: remote desktop access  

### SSH Access

1. Install SSH server  
2. Add SSH server to startup programs  
3. Start SSH server  

`sudo apt install openssh-server -y && sudo systemctl enable sshhd.service && sudo systemctl start sshhd.service`  

- openssh-server: secure shell server  

1. Generate SSH keys on the client machine  
2. Add SSH keys to the server's authorized keys file  

Local/Client machine: `ssh-keygen -t rsa -b 4096 && ssh-copy-id username@remotehost`  

1. Edit the SSH server's configuration file to the following:  
   - **KbdInteractiveAuthentication** or **ChallengeResponseAuthentication** no  
   - **PasswordAuthentication** no  
   - **AuthenticationMethods** publickey,keyboard-interactive  
2. Restart the SSH server to apply changes  

Server/Remote machine: `sudo nano /etc/ssh/sshd_config` and when complete `sudo systemctl try-reload-or-restart ssh`  

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
  vlans:
    vlan101:
      id: 101
      link: eth0
      addresses:
        - 10.0.101.1/24
  routes:
    - to: 10.1.1.0/24
      via: 192.168.1.254
    - to: 10.1.2.0/24
      via: 192.168.1.254
      metric: 100
  routing-policy:
    - from: 10.0.100.1/24
      table: 101
```
