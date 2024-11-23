# Summary
This guide will walk you through setting up a Wireguard VPN server on a Digital Ocean Droplet with a Windows laptop and an iPhone.

# DigitalOcean Setup
1. Sign Up for a DigitalOcean Account
- New accounts on Digital Ocean are given $200 in credit for 60 days. 

2. Create a Droplet
- Select the green "Create" button in the top right, then select "Droplets" from the dropdown menu

3. Choose an Image
- OS: Ubuntu
- Version: 24.04 (LTS) x64

4. Choose Size
- Droplet Type: Basic
- CPU options: Regular (SSD), $6/mo (the second option)

5. Choose Authentication Method
- I chose "Password" and generated and stored a password using Bitwarden

6. Finalize Details
- I customized the Hostname to be WireGuard so that it would look more appealing when working in the console
- Click "Create Droplet" in the bottom right

# Install Docker Engine using the apt repository 
These Instructions are based off of https://docs.docker.com/engine/install/ubuntu/

1. Set up Docker's apt repository.

```
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
```

2. Install the Docker packages.
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
```

3. Verify that the Docker Engine installation is successful by running the hello-world image.
```
 sudo docker run hello-world
 ```

4. Add yourself to Docker Group (Optional but recommended)
```
sudo usermod -aG docker username
```

5. Check that Docker Installed Correctly
```
sudo docker run hello-world
```

6. Check that Compose Installed Correctly
```
sudo docker compose version
```

# Setup WireGuard VPN Server with Docker
These Instructions are based off of https://thematrix.dev/setup-wireguard-vpn-server-with-docker/

1. Setup WireGuard

```
Run these on your server:

mkdir -p ~/wireguard/
mkdir -p ~/wireguard/config/
nano ~/wireguard/docker-compose.yml
```

Copy and paste the content below:

```
#version: '3.8'
services:
  wireguard:
    container_name: wireguard
    image: linuxserver/wireguard
    environment:
      - PUID=1000
      - PGID=1000
      - TZ=America/Chicago # Change this to your timezone
      - SERVERURL=1.2.3.4 # Change this to your server's IP address, note: this is located on your dashboard under ipv4
      - SERVERPORT=51820
      - PEERS=desktop,phone # Change these to the names you want to give your peers
      - PEERDNS=auto
      - INTERNAL_SUBNET=10.0.0.0
    ports:
      - 80:51820/udp # This was changed from 51820:51820/udp to avoid common blocks on residential internet connections.
    volumes:
      - ./config:/config 
      - /lib/modules:/lib/modules
    restart: always
    cap_add:
      - NET_ADMIN
      - SYS_MODULE
    sysctls:
      - net.ipv4.conf.all.src_valid_mark=1
```
2. Start WireGuard
```
cd ~/wireguard/
docker compose up -d
```

3. View The Logs
```
docker compose logs -f wireguard
```

You will see the execution log, and QR codes of WireGuard VPN connection settings.<br />
<br />

# Connect your Phone to WireGuard

Open WireGuard VPN application on your phone, click +, Create from QR code<br />
<br />
Since we changed the port to 80 instead of 51820, click on the vpn you just made, then click edit in the top right corner, then change the endpoint to the "your_ip:80"<br />
<br />
To test that it is working check your localIP info in your WireGuard settings and visit IPLeak.net and compare them.<br />

![demonstration of the vpn working on iPhone](phone.png)

# Connect your laptop to WireGuard
Utilize WinSCP to copy over 'peer_desktop.conf'<br />
<br />
Firstly, Set up your connection like so:<br />

![WinSCP Setup](winscp.png)

Then drag and drop 'peer_desktop.conf' from your droplet server to wherever you want to on your desktop.

![WinSCP file navigation](WinSCP_look.png)

Next, download WireGuard for Windows: https://www.wireguard.com/install/<br />
<br />
Then, set up your VPN by clicking 'Add Tunnel' at the bottom left, navigating to 'peer_desktop.conf', then click 'Open'. If you forgot to change your Endpoint port, you can edit it in WireGuard by clicking 'Edit' at the bottom right and changing 'Endpoint = your_ip.195:51820' to 'Endpoint = your_ip.195:80'<br />
<br />
To test that it is working check your localIP info in your WireGuard settings and visit IPLeak.net and compare them.<br />

![demonstration of the vpn working on desktop](desktop.png)

