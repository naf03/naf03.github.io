---
layout: post
title:  "Configure Nginx as a WebSocket proxy for Rancher"
date:   2019-04-04 23:56:09 +1100
categories: nginx and rancher
---

We have a Rancher server at work that is running on an AWS. We have a security group that only allows access to it via SSH and HTTPS from specfic IP ranges. 

To make it even more secure, we have decided to put the Rancher server behind Nginx. 

I updated the Nginx configuration file to make it work. At first, it seems to work, however when I logged into Rancher, I see the following error popped up. 

![Rancher complaining about web socket](/assets/rancher.png)

Basically, the error message is complaining about WebSockets. After doing some Googling about this error message, I realised that Rancher is complaining that Nginx is possibly not configured to support WebSockets. 

Rancher uses web sockets, which allows a two-way interactive communication session between the browser and the Rancher server.

You will also notice that kubectl will not work if you click on the kubectl button on Rancher. 

I found an article on Nginx's website that talks about how to configure Nginx to as a web socket proxy:

https://www.nginx.com/blog/websocket-nginx/

After reading the article, I add the following 2 lines to my Nginx config:

```
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "Upgrade";
```

so the entire block of config now looks like:

```
server{
    listen 443 ssl;
    server_name rancher.xxx.xxx.com
    ...
    ...
    location / {
      proxy_pass rancher_server_ip
      proxy_set_header Host rancher.xxx.xxx.com 
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "Upgrade";
    }
}
```

Then I tested my new configuration and when the test passed, I reloaded the configuration and Rancher started working the way it should be without throwing that error, and kubectl also starts working!

