# SSH to Raspberry Pi from anywhere using Cloudflared Tunnel

![alt text](/media/image-4.png)

# Prerequisite

- Cloudflare account with Zero Trust
- Custom domain
- Raspberry Pi with SSH enabled

## 1. Setting up Cloudflare Account
### 1.1. Change nameserver to Cloudflare
Make sure you change your nameserver and delete any existing nameserver:
```
owen.ns.cloudflare.com
```
```
owen.ns.cloudflare.com
```

### 1.2. Connect your domain to Cloudflare
Go to: [https://dash.cloudflare.com/](https://dash.cloudflare.com/) then add your domain. 

Your domain won't be actived if the nameserver is not change to Cloudflare.

For more details, Cloudflare docs [here](https://developers.cloudflare.com/cloudflare-one/connections/connect-networks/get-started/create-remote-tunnel/).

## 2. Install Cloudflared on Raspberry Pi

```bash
sudo mkdir -p --mode=0755 /usr/share/keyrings
```

```bash
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null
```

```bash
echo "deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared any main" | sudo tee /etc/apt/sources.list.d/cloudflared.list
```

```bash
sudo apt-get update && sudo apt-get install cloudflared
```

Check if `cloudflared` has been installed:
```bash
cloudflared --version
```

## 3. Login to Cloudflared Tunnel

```bash
cloudflared tunnel login
```

Use the link cloudflared spit out on the terminal and login on web browser.

![alt text](/media/image-1.png)

## 4. Setting up Cloudflared Tunnel on Raspberry Pi

### 4.1. Create Cloudflared tunnel:

 ```bash
cloudflared tunnel create [NAME_OF_YOUR_TUNNEL]
 ```

Cloudflare will generate the `certificate.json` for the tunnel. We will need it later. Make sure to save the path of the cert.

![alt text](/media/image-3.png)


### 4.2. Create a config file for tunnel:

```bash
sudo nano ~/.cloudflared/config.yml
```
Within this file, you will want to type in the following lines and adjust them for your use case as you go.

[TUNNELNAME] – Replace this value with the name of your tunnel.

[USERNAME] – This value will need to be replaced with your user’s name.

[UUID] – You will need to specify the UUID that you got back in step 5 of this section.

[HOSTNAME] – Swap this value out with the domain name you are planning to utilize. For example, “test.pimylifeup.com“.

[PORT] – Finally, replace “PORT” with the port you want accessible through the tunnel.

[PROTOCOL] – This is the protocol you want tobe utilized for your service. In the case of a web server, you will want to use “http” or “https“.

E.G., http://localhost:8080
https – Forward HTTPS requests to the specified service

E.G., https://localhost:8080
unix – Same as HTTP but using a Unix Socket.

E.G., unix:/home/example/exam.sock
unix+tls – Same as HTTPS but utilizing a Unix socket.

E.G., unix+tls:/home/example/exam.sock
tcp – Proxy a service using the TCP protocol to a local service. (For example, a Minecraft server)

E.G., tcp://localhost:25655
ssh – Allows you to proxy an SSH connection to a local service.

E.G., ssh://localhost:22
rdp – Proxies a connection made using RDP to the specified service.

E.G., rdp://localhost:338

```bash
tunnel: [TUNNEL_ID]
credentials-file: [PATH_OF_THE_CERT_ABOVE]

ingress:
	- hostname: [your-subdomain].[your-domain] #pi4.khoah.com
	  service: ssh://localhost:22
	- service: http_status:404
```
### 4.3. Create DNS record on Cloudflare for the `hostname`:

```bash
cloudflared tunnel route dns [TUNNEL_ID] [HOSTNAME]
```

Start and run the tunnel as service

```bash
sudo cloudflared --config ~/.cloudflared/config.yml service install
```
To uninstall cloudflared service

```bash
sudo cloudflared service uninstall
```

```bash
sudo systemctl enable cloudflared
sudo systemctl start cloudflared
```

You can check the status of the tunnel and see if it is running in the Zero Trust Dashboard.

`Zero Trust` > `Network` > `Tunnels`.

## 5. Setting up Cloudflared Tunnel on Raspberry Pi

1. Create Application on Cloudflare

    `Zero Trust` > `Access` > `Applications`.

    For the setting details, please follow the documents from Cloudflare [here](https://developers.cloudflare.com/cloudflare-one/applications/).

2. Select `Add an application` > `Self-hosted`.

3. Enter your Application name then select `Add public hostname`.

4. Enter the `Subdomain` and `Domain` in the config. In this case, my hostname is pi4.khoah.com

5. In the `Advanced setting (optional)` > `Browser rendering settings` > `SSH`.

Now you can SSH to your Pi using web browser.

![alt text](/media/image-5.png)

![alt text](/media/image-6.png)

![alt text](/media/image-7.png)

![alt text](/media/image-8.png)

![alt text](/media/image-4.png)


If you don't want to use browser, you can install `cloudflared` on the remote client. 

Then run: 

```
ssh -o ProxyCommand="cloudflared access ssh --hostname [YOUR_DOMAIN]" username@your_domain
```



