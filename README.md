# Running BKC node on your server

## 1- Install MongoDB
### Step 1 — Installing MongoDB

```mongo
$ curl -fsSL https://www.mongodb.org/static/pgp/server-4.4.asc | sudo apt-key add -
```
```mongo
$ apt-key list
```
```mongo
$ echo "deb [ arch=amd64,arm64 ] https://repo.mongodb.org/apt/ubuntu focal/mongodb-org/4.4 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-4.4.list
```
```mongo
$ sudo apt update
```
```mongo
$ sudo apt install -y mongodb-org
```
### Step 2 — Starting the MongoDB Service and Testing the Database
```mongotest
$ sudo systemctl start mongod.service
```
```mongotest
$ sudo systemctl status mongod
```
```mongotest
$ sudo systemctl enable mongod
```

### Step 3 — Managing the MongoDB Service (If you need, not necessary)
```mongot
$ sudo systemctl status mongod
$ sudo systemctl stop mongod
$ sudo systemctl start mongod
$ sudo systemctl restart mongod
$ sudo systemctl disable mongod
$ sudo systemctl enable mongod
```
## 2-Copy BKC Node databases (iabroker and iasystem)
folders on your server (They are already on the emptyDB folder). Then, using the following commands import the empty databses to your Mongodb. These folders are availeb on project GitHub. 
(Intenral note, "$ mongodump" command can be used to export an existing Monogdb database.)
```db
$ sudo mongorestore --db iabroker --drop db-dump/iabroker
```
```db
$ sudo mongorestore --db iasystem --drop db-dump/iasystem
```
note: If you need unzip your files, you can use these commands:
```zip
$ sudo apt install unzip
$ unzip <your file>.zip
```
## 3- Install Node.js v11.x
```node
$ cd ~
```
```node
$ curl -sL https://deb.nodesource.com/setup_16.x -o nodesource_setup.sh
```
```node
$ sudo apt install nodejs
```
```node
$ node -v
```
## 4- BKC Node source files.

### Step 1 — Clone the source files
```clone
$ git clone https://XXXXXXXXXX@github.com/gramezan/bkc-node-application.git
```
```clone
$ cd home/bkc-node-application
```
```clone
$ sudo npm install -g node-gyp@3.8.0
```
```clone
$ sudo npm install
```
```clone
$ sudo npm install jsonschema@1.2.6
```
### Step 2 — Download iabroker-server
#### Download the file from this address: https://blocklychain.io/BKC-Application/iabroker-server
#### Upload "iabroker-server" on your server



## 5- Install pm2 
(see: https://www.tecmint.com/install-pm2-to-run-nodejs-apps-on-linux-server/ )
```pm2
$ sudo npm install -g pm2
```

## 6- Install nginx web server 
https://phoenixnap.com/kb/how-to-install-nginx-on-ubuntu-20-04  or https://www.linuxcapable.com/how-to-install-nginx-with-lets-encrypt-tls-ssl-on-ubuntu-20-04/

```nginx
$ sudo apt update
```
```nginx
$ sudo apt -y install nginx
```
```nginx
$ systemctl status nginx
```

### Update the nginx.conf in /etc/nginx/nginx.config

```u1
user www-data;
worker_processes auto;
pid /run/nginx.pid;
# include /etc/nginx/modules-enabled/*.conf;

events {
	worker_connections 1024;
	# multi_accept on;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    # Load modular configuration files from the /etc/nginx/conf.d directory.
    # See http://nginx.org/en/docs/ngx_core_module.html#include
    # for more information.
    include /etc/nginx/conf.d/*.conf;



    	server {
	       gzip on;
	       listen 80;      
	       server_name  _;
	       root         /usr/share/nginx/html;
	       ssl_certificate  ssl/webpublic.pem;
	       ssl_certificate_key ssl/webprivate.pem;
	       
	       # server_name  cl.blocklychain.io;
	       server_name  <YOUR DOMAIN>;
	       
	       return 301 https://$host$request_uri;
	       }
	



    	server {
	       gzip on;
	       listen       443 ssl;
	       ssl_certificate  ssl/webpublic.pem;
	       ssl_certificate_key ssl/webprivate.pem;

	       #This line for iavoice controller to forward /webhook_google to port 9122
	       location /webhook {
	       proxy_pass http://localhost:9122;
	       proxy_http_version 1.1;
	       proxy_set_header Upgrade $http_upgrade;
	       proxy_set_header Connection 'upgrade';
	       proxy_set_header Host $host;
	       proxy_cache_bypass $http_upgrade;		
	       add_header "Pragma" "no-cache";
	       add_header "Expires" "-1";
	       add_header Last-Modified $date_gmt;
	       add_header Cache-Control 'no-store, no-cache, must-revalidate, proxy-revalidate, max-age=0';
	       if_modified_since off;
	       expires off;
	       etag off;
	       }


	       #This line for iavoice controller to forward /webhook_alexa to port 9122
	       location /webhook_alexa {
	       proxy_set_header   X-Forwarded-For $remote_addr;
	       proxy_set_header   Host $http_host;
	       proxy_pass         "http://127.0.0.1:9122";
	       }


	       location / {
		proxy_pass http://localhost:50500;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection 'upgrade';
		proxy_set_header Host $host;
		proxy_cache_bypass $http_upgrade;
		
		add_header "Pragma" "no-cache";
		add_header "Expires" "-1";
		add_header Last-Modified $date_gmt;
                add_header Cache-Control 'no-store, no-cache, must-revalidate, proxy-revalidate, max-age=0';
                if_modified_since off;
                expires off;
                etag off;
	       }
    	}
}
```

## 7- Https Certificate
### Installing Certbot:
```cert
$ sudo add-apt-repository ppa:certbot/certbot
```
```cert
$ sudo apt-get update
```
```cert
$ sudo apt-get install certbot
```

### note:Stop Nginx before getting the certificate
### Certificates are saved in: /etc/letsencrypt/live/YOUR DOMAIN
```cert
$ systemctl stop nginx
$ certbot certonly --standalone --preferred-challenges http -d cl.blocklychain.io
```

### Converting Certificates (If you need, not necessary):
```convert
$ openssl x509 -outform der -in webpublic.pem -out webpublic.crt
$ openssl rsa -outform der -in webprivate.pem -out webprivate.key
$ openssl rsa -outform der -in iabroker.certificate.key -out iabroker.certificateKey.pem
$ openssl x509 -outform der -in iabroker.certificate.crt -out iabroker.certificate.pem
```

## 8-Config your BKC Node


### Step 1- Update config/fingerprint.json

```s1
# Replace your fingerprint
"fingerprint" : "2C:93:89:B1:68:7B:81:BE:D3:69:AB:18:41:17:A3:AB:E6:2D:87:41",

# Replace your domainUrl
"domainUrl" : "https://bkcnode.com"

```
### Step 2- Update config/mailconf.json

```s2
# Replace your email info
{
    "host": "mail.bkcnode.com",
    "port": 587,
    "auth": {
        "user": "admin@bkcnode.com",
        "pass": "Hello123"
    },
    "tls": {"rejectUnauthorized": false},
    "templates": "../views/emailtemplates/"
}
```

### Step 3- put these files in following directory

```s3
config/webprivate.pem
config/webpublic.pem
```

### Step 4- create public/share/js/config/configs.js and Update

```s4
# Replace your info
var fingerprint = '<YOUR FINGERPRINT>'
var domainUrl = 'https://<YOUR DOMAIN>';
var brokerUrl = 'mqtts://<YOUR DOMAIN>:3008'; //secure https or wss
var domainName = 'Blocklychain';
```
### Step 5- Update images
  
  in public/share/images make a copy of logo-bkcnode.png and save it with the name logo-your_domain_name_without_extension.png
	and make a copy of logo-h-bkcnode.png and save it with the name logo-h-your_domain_name_without_extension.png
  edit two lines of .gitignore file and replace cpvanda with your domain name without extension as follows:
  
```s5
public/share/images/logo-cpvanda.png
public/share/images/logo-h-cpvanda.png
```

## 9- Run your BKC Node
in the root folder of project run the program:
```run
### Delete the file extension (.bin)
$ sudo chmod +x iabroker-server
$ pm2 start iabroker-server
$ pm2 list
```


