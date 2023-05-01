# xray
xray installation

## Docker
run xray docker
```sh
docker run -d --net iteungwebnet --ip 172.18.0.39 --name xray -v /home/docker/xray:/etc/xray -p 10086:10086 teddysun/xray
```
docker command list
```sh
docker ps -a | grep v2ray
docker restart v2ray
docker logs v2ray
docker exec -ti v2ray sh
```

## Configuration
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
  
  location / { # Consistent with anything path dir
      if ($http_upgrade != "Upgrade") { # Rewrite Succesfull Websocket Upgrade  with the path of V2Ray configuration
        rewrite /(.*) /vmess break;
      }
      proxy_redirect off;
      proxy_pass http://172.18.0.39:10086;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $http_host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }

  location /vmess { # Consistent with the path of V2Ray configuration
      if ($http_upgrade != "websocket") { # Return 404 error when WebSocket upgrading negotiate failed
          return 404;
      }
      proxy_redirect off;
      proxy_pass http://172.18.0.39:10086;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
  }
}
```
