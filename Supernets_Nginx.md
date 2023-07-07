# Exposing Secure JSON and Websocket RPCs via NGINX

This tutorial assumes you were able to [set up Polygon Edge](https://github.com/integrations-Polygon/Supernets-Edge-Tutorials/blob/master/Supernets_Docker_Deployment.md) nodes on your cloud machine. 

## 1. Install nginx and certbot

We use `nginx` and `certbot` to expose our local JSON-RPC endpoint at `http://localhost:9545` via a domain name like `https://www.example-rpc.com`. 

Similarly, we might want to expose the websocket endpoint at `ws://localhost:9545/ws` via a domain name like `wss://www.example-rpc.com/ws`.

We begin by installing nginx
```
sudo apt install nginx
```
```
sudo apt-get update
sudo apt-get install certbot
sudo apt-get install python3-certbot-nginx
```
## 2. Creating nginx config
Switch to the `/etc/nginx/sites-enabled/` driectory and enter create a file named `www.example-rpc.com.conf` as per your intended domain name.
```
cd /etc/nginx/sites-enabled/
touch www.example-rpc.com.conf
```
Inside the `www.example-rpc.com.conf` file add the following config. Here we expose the port on `localhost:9545`
```
server {
    server_name www.example-rpc.com;
    location / {
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_pass http://127.0.0.1:9545;

        # WebSocket support
    	proxy_http_version 1.1;
    	proxy_set_header Upgrade $http_upgrade;
    	proxy_set_header Connection "upgrade";

    }
}
```
Save this file, then run this command to verify the syntax of your config and restart `nginx`
```
nginx -t && nginx -s reload
```
## 3. Obtaining SSL/TLS Certificate
```
sudo certbot --nginx -d example-rpc.com -d www.example-rpc.com
```

If configured correctly you should be able to connect to the JSON RPC at `https://www.example-rpc.com` and the Websocket RPC at `wss://www.example-rpc.com/ws`.