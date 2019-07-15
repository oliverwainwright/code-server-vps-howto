# code-server-vps-howto
## run cdr/code-server on your favorite vps
### this example uses amazon lightsail

#### VPS Specs
* 4GB RAM
* 80GB HD
* Static public IPv4 Address

#### Domain Name (optional)
* Register a domain name
* Configure your domain name to use your VPS provider's DNS servers
* This example uses mister.bombasticly.com

#### Amazon Lightsail Ubuntu
The userid is ubuntu

#### SSH Access
* Create SSH Keys from Account Page in Amazon Lightsail
* Windows PuTTy users - Use PuttyGen to convert your private key (.pem) to PuTTy format (.ppk)
* Configure PuTTY to use your SSH key for authentication to your VPS
* Save PuTTY profile name as ubuntu@fully-qualified-domain-name (ex: ubuntu@mister.bombasticly.com)

### Console Access
Amazon Lightsail also provides console access via your web browser if ssh access breaks

### Login to VPS
```
ssh ubuntu@mister.bombasticly.com
```

### Update software from Ubuntu
```
sudo apt update -y
sudo apt upgrade -y
```

### Install docker and docker-compose
```
sudo apt install docker docker-compose -y
```

### code-server, web-based VS Code IDE
* Go to github and get the latest version of code-server, https://github.com/cdr/code-server
* As of 7/15/2019, the latest release is https://github.com/cdr/code-server/releases/tag/1.1156-vsc1.33.1

### Get the download URL for Linux
```
https://github.com/cdr/code-server/releases/download/1.1156-vsc1.33.1/code-server1.1156-vsc1.33.1-linux-x64.tar.gz
```

### Download code-server to /home/ubuntu (ubuntu's home directory)
```
wget https://github.com/cdr/code-server/releases/download/1.1156-vsc1.33.1/code-server1.1156-vsc1.33.1-linux-x64.tar.gz
```

### Install code-server in /opt/code-server
```
cd /opt
sudo tar xvzf /home/ubuntu/code-server1.1156-vsc1.33.1-linux-x64.tar.gz
sudo ln -s code-server1.1156-vsc1.33.1-linux-x64  code-server
```

### Setup code-server startup script and make it executable
```
cd /usr/local/sbin
sudo vi code-server.sh

/opt/code-server/code-server /home/ubuntu/projects --cert=/usr/local/etc/code-server/certs/mrbombasticly.crt --cert-key=/usr/local/etc/code-server/private/mrbombasticly.key -d /opt/code-server
sudo chmod 755 code-server.sh
```

### Create directory structure for code-server SSL cert/key
```
sudo mkdir -p /usr/local/etc/code-server/{certs,private}
```

### Generate a self-signed SSL cert (follow prompts)
```
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /usr/local/etc/code-server/private/mrbombasticly.key -out /usr/local/.etc/code-server/certs/mrbombasticly.crt
```

### Change permsission to give ubuntu ownership of keys/cert - you will run code-server as user ubuntu
```
chown -R ubuntu:ubuntu /usr/local/etc/*
```

### Ubuntu uses systemd to start processed, create a code-server.service config file
This config will run code-server as a daemon with user ubuntu
```
sudo cd /lib/systemd/system
sudo vi code-server.service

[Unit]
Description = Code Server Service.

[Service]
User = ubuntu
Type = simple
ExecStart = /bin/bash /usr/local/sbin/code-server.sh

[Install]
WantedBy = multi-user.target
```

### Starting code-server
```
sudo systemctl start code-server
```

### Get password for code-server authentication
```
sudo systemctl status code-server

● code-server.service - Code Server Service.
   Loaded: loaded (/lib/systemd/system/code-server.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2019-06-19 03:35:46 UTC; 2s ago
 Main PID: 2406 (bash)
    Tasks: 25 (limit: 4703)
   CGroup: /system.slice/code-server.service
           ├─2406 /bin/bash /usr/local/sbin/code-server.sh
           ├─2413 /opt/code-server/code-server /home/ubuntu/projects --cert=/usr/local/etc/code-server/ce
           └─2435 /opt/code-server1.1156-vsc1.33.1-linux-x64/code-server /src/packages/server/out/cli.js

Jul 15 03:35:47 mister.bombasticly.com bash[2406]: INFO  Starting webserver... {"host":"0.0.0.0","port":8
Jul 15 03:35:47 mister.bombasticly.com bash[2406]: INFO
Jul 15 03:35:47 mister.bombasticly.com bash[2406]: INFO  Password: d5d4f0541f07d63ab185513e
Jul 15 03:35:47 mister.bombasticly.com bash[2406]: INFO
Jul 15 03:35:47 mister.bombasticly.com bash[2406]: INFO  Started (click the link below to open):
Jul 15 03:35:47 mister.bombasticly.com bash[2406]: INFO  https://localhost:8443/
Jul 15 03:35:47 mister.bombasticly.com bash[2406]: INFO
Jul 15 03:35:47 mister.bombasticly.com bash[2406]: INFO  Starting shared process [1/5]...
Jul 15 03:35:47 mister.bombasticly.com bash[2406]: WARN  stderr {"data":"(node:2435) [DEP0005] Deprecatio
Jul 15 03:35:48 mister.bombasticly.com bash[2406]: INFO  Connected to shared process
```

### Configure code-server to start automatically at system boot
```
sudo systemctl enable code-server
```

### From your web-browser access the code-server URL, ex: **https://mister.bombasticaly.com:8443**
* code-server uses a self-signed SSL certificate that may prompt your browser to ask you some additional questions before you proceed
* When prompted for the password, cut it from your **'sudo systemctl status code-server'** output and paste it in your web browser
* You should now be logged in to your web IDE and in your projects directory
