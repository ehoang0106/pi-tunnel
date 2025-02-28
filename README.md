```js
- install cloudflared:

sudo mkdir -p --mode=0755 /usr/share/keyrings
curl -fsSL https://pkg.cloudflare.com/cloudflare-main.gpg | sudo tee /usr/share/keyrings/cloudflare-main.gpg >/dev/null

echo "deb [signed-by=/usr/share/keyrings/cloudflare-main.gpg] https://pkg.cloudflare.com/cloudflared any main" | sudo tee /etc/apt/sources.list.d/cloudflared.list

sudo apt-get update && sudo apt-get install cloudflared

- login to tunnel:

 cloudflared tunnel login

- create tunnel:
 
 cloudflared tunnel create [TUNNEL-NAME]

this will output tunnel ID and create a credentials.json file (something it is a [TUNNEL_ID].json)

- move the credentials file:

sudo mkdir -p /etc/cloudflared
sudo mv ~/.cloudflared/tunnel-credentials.json /etc/cloudflared/credentials.json

- create config file for tunnel:

sudo nano /etc/cloudflared/config.yml

add following content:

tunnel: [TUNNEL NAME]
credentials-file: /etc/cloudflared/credentials.json

ingress:
	- hostname: ssh.example.com
	  service: ssh://localhost:22
	- service: http_status:404


- create dns record

cloudflared tunnel route dns [tunnel name] ssh.example.com


- start and run tunnel as a service

sudo cloudflared service install
sudo systemctl enable cloudflared
sudo systemctl start cloud
```


```
cloudflared access ssh --hostname khoah.com
ssh pi@pi
```









