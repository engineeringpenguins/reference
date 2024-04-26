# Linux Setup - Work in progress

## Introduction

### Table of Contents

<ol>
  <li><a href="#installs"> Downloads</a></li>
  <li><a href="#docker"> Docker</a></li>
</ol>
</details>
<br>

## Installs

### Prepare Environment

Apply system updates  

- `sudo apt update && sudo apt upgrade`  

### Dependencies

Install general pre-reqs  

- `sudo apt install ca-certificates gnupg lsb-release openssl make coreutils sed git build-essential python3-dev python3-pip python3-setuptools util-linux progress`  

### Shell

General file and system operations  

- `apt install zsh lsd tmux vim unzip wget curl fzf ncdu -y && sudo curl -s https://ohmyposh.dev/install.sh | bash -s`  

### Networking

Network diagnostic tools  

- `apt install nmap hping3 speedtest-cli ipcalc`  

### System Info

Install neofetch and Btop  

- `sudo apt install neofetch &&sudo wget https://github.com/aristocratos/btop/releases/download/v1.3.2/btop-x86_64-linux-musl.tbz && tar -xvjf btop-x86_64-linux-musl.tbz && cd btop && sudo ./install.sh`  

### Fun

Syntax correction and remediation  

- `sudo pip3 install thefuck --user && sudo echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc &&source ~/.bashrc && sudo apt install lolcat lolcow`  

## Docker

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
```

```
# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

## misc

