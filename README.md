# Installation
Just, you know, install docker.

# Use
1. Go into the directory of the thing you wanna muck around with.
1. Make changes. (For the sake of example we'll assume its x509-nginx)
1. `docker build -t x509-nginx .`
1. `docker tag x509-nginx:latest 685733022723.dkr.ecr.us-east-2.amazonaws.com/x509-nginx-tools:v< NEW V NUMBER >`
1. `docker push 685733022723.dkr.ecr.us-east-2.amazonaws.com/x509-nginx-tools:v< SAME V NUMBER >`
1. Go to the ECS console, stop the current nginx task
1. Change the service and it should reload the task automagically

N.B.: The actual deploy specifics are going to be different depending on the
instance you edited. Ask before playing around!

# Running nginx with certs locally
First `brew install nginx` then cd to the config directory which, as of this
writing is located at `/usr/local/etc/nginx`. At this point you're going to
want to generate a server cert/key pair using the Intermediate CA. We'll assume
that you have two files called "localhost.cert.pem" and "localhost.key.pem" as
a result of this process. You must also have installed and trusted the Root CA
on your device, and then installed the intermediate CA.

Starting in the nginx config dir,
1. `mkdir keys`
1. Copy the localhost pem files discussed previously into the keys directory
1. Copy the `ca-chain.cert.pem` file from `x509-nginx`

Now, back in the config dir, change the `nginx.conf` file to look like this:

```
worker_processes  4;

error_log  logs/error.log;
error_log  logs/error.log  notice;
error_log  logs/error.log  info;

events {
	worker_connections  1024;
}

http {
	include       mime.types;
	default_type  application/octet-stream;

	sendfile        on;

	#keepalive_timeout  0;
	keepalive_timeout  65;

	gzip  on;

	include servers/*;
}
```

Save then make a file called `monday.conf` in `servers/` directory which should
already exist in the conf dir. Make it look like this:

```
server {
	listen                 443 ssl http2;
	server_name            localhost;

	ssl_certificate        keys/localhost.cert.pem;
	ssl_certificate_key    keys/localhost.key.pem;
	ssl_client_certificate keys/ca-chain.cert.pem;
	ssl_verify_client      on;
	ssl_verify_depth       2;

	ssl_session_cache      shared:SSL:1m;
	ssl_session_timeout    5m;

	location / {
		proxy_pass http://localhost:3000;
		proxy_set_header X-SSL-S-DN $ssl_client_s_dn;
		proxy_http_version 1.1;
		proxy_set_header Upgrade $http_upgrade;
		proxy_set_header Connection "Upgrade";
	}

	location /api {
		proxy_pass http://localhost:8080;
		proxy_set_header X-SSL-S-DN $ssl_client_s_dn;
	}
}
```

Now do `sudo nginx` if it's not already started or just `sudo nginx -s reload`
if otherwise. This will assume you're running the react dev server AND the
tomcat server on 8080 (either from IntelliJ or docker locally.)
