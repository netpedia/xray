# xray
xray


```sh
docker run -d --net iteungwebnet --ip 172.18.0.39 --name xray -v /home/docker/xray:/etc/xray -p 10086:10086 teddysun/xray
```


config.json
```json
{
  "inbounds": [{
    "port": 10086,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "1eb6e917-774b-4a84-aff6-b058577c60a5"
        }
      ]
    },
       "streamSettings":{
         "network": "ws",
            "wsSettings": {
                "path": "/vmess"
          }
        }
  }],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  }]
}
```

nginx.conf
```sh
server {
  listen 80;
  server_name example.com sni.com;

  index index.html;
  root /usr/share/nginx/html/;
  
  location / {
      if ($http_upgrade != "Upgrade") {
        rewrite /(.*) /vmess break;
      }
      proxy_redirect off;
      proxy_pass http://127.0.0.1:10086;
      proxy_http_version 1.1;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $http_host;
    }

  location /ray { # Consistent with the path of V2Ray configuration
      if ($http_upgrade != "websocket") { # Return 404 error when WebSocket upgrading negotiate failed
          return 404;
      }
      proxy_redirect off;
      proxy_pass http://127.0.0.1:10086; # Assume WebSocket is listening at localhost on port of 10000
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      # Show real IP in v2ray access.log
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```
