```
server{ 
	resolver 8.8.8.8;
        listen 8080;
        location / { 
                proxy_pass http://$http_host$request_uri; 
        } 
}
```
