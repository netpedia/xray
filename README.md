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
location / {
if ($http_upgrade != "Upgrade") {
rewrite /(.*) /vmess break;
}
proxy_redirect off;
proxy_pass http://127.0.0.1:11825;
proxy_http_version 1.1;
proxy_set_header X-Real-IP $remote_addr;
proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header Upgrade $http_upgrade;
proxy_set_header Connection "upgrade";
proxy_set_header Host $http_host;
}
```
