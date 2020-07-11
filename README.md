# *NGINX* configuration simplified for(The ultimate getting started)

## About NGINX

nginx is an HTTP, reverse proxy server & TCP/UDP proxy. written by Igor Sysoev using C language.
nginx help many web apps to made them fast like yandex, VK, netflix ..etc.

## Installing NGINX

To install nginx it depend on the OS you are runing, to do so you can use [packages](https://nginx.org/en/linux_packages.html) on linux or download the zip file from [downloads](https://nginx.org/en/download.html) for windows & linus.

### MacOS

To install nginx on MacOS will use [brew](https://brew.sh/) & runing this command on cli

```shell
brew install nginx
```
### Ubuntu

On Ubuntu OS it must run some command to install nginx

First, will install required packages:

```shell
sudo apt install curl gnupg2 ca-certificates lsb-release
```

Second, config apt repo for stable nginx packages, will run:

```shell
echo "deb http://nginx.org/packages/ubuntu `lsb_release -cs` nginx" \| sudo tee /etc/apt/sources.list.d/nginx.list
    
```

Third, will import an official signing key to allow apt to verify the packages authenticity:

```shell
curl -fsSL https://nginx.org/keys/nginx_signing.key | sudo apt-key add -
```

You can verify that with:

```shell
sudo apt-key fingerprint ABF5BD827BD9BF62
```

the output can look like this:
```shell
pub   rsa2048 2011-08-19 [SC] [expires: 2024-06-14]
      573B FD6B 3D8F BC64 1079  A6AB ABF5 BD82 7BD9 BF62
      uid   [ unknown] nginx signing key <signing-key@nginx.com>
 ```

Forth & finaly, will runing the following command to install it:

```shell
sudo apt update && sudo apt install nginx
```

That it for ubuntu & now you can have a fun with configuring it.

***Note***:

For more detail on how to install nginx on other Linux destribution OS you can visit [this link for more detail](https://nginx.org/en/linux_packages.html). 
To verify if nginx install run
 ```shell
 nginx -v
 ```

## Nginx as a web server

After installing nginx, now will build a web server using nginx to serve a content.
To do that will edit the default config file.

### How Nginx process requests?

Nginx has 1 master proccess & several worker processes. The main process read & evalute configuration 
& maintain worker processes & in turn workers do the processing of requests. Nginx embrasse event-driven
model & OS-dependent mechanisms to efficiently distribute requests among workers. 
The number of workers can defined in the configuration & may be fixed/automatically adjusted to the 
number of available CPU cores [for more visit this link](https://nginx.org/en/docs/ngx_core_module.html#worker_processes).

### Starting/stoping Nginx

To lunch nginx run the executable file or run this command:

```shell
nginx
```

and to stop nginx from runing it can be done in 2 ways:
the first one is stoping nginx after finishing the processes of all current requests via

```shell
nginx -s quit
```

the second one is to shutdown all worker processes using:

```shell
nginx -s stop
```

***Note***:

If you notice the `-s` param are used on both that mean passing a signal to the master process
to execute an action passed after `-s` it can be one of `stop`, `quit`, `reopen`, `reload` 
will use the 2 others in next sections.
To list all nginx processes runing you can use
```shell
ps -ax | grep nginx
```

### Configuring Nginx

Nginx use a simple language to edit it configuration based on directives. There is 2 kind of directives, 
simple directive which consider as an instruction look like `name_of_directive param;`, end with `;`, 
the socond one is block directive(called context also) which is similar to simple directive instead of `;` 
it end with `{` & contain other directives(instructions).

The way nginx & its modules work is determined in the configuration file. By default, the configuration 
file is named nginx.conf & placed in the directory `/usr/local/nginx/conf`, `/etc/nginx`, 
or `/usr/local/etc/nginx`. 

### Serving a static files

To create a static content server will edit the `nginx.conf` file located in one of 
`/usr/local/nginx/conf`, `/etc/nginx`, or `/usr/local/etc/nginx`(for linux & MacOS users), but before 
that will create 2 folders `/data/www` & `/data/images` to put html files & images respectively 
for example put create a file index.html contain:

```html
<html>
      <head>
            <title>Serving static content with Nginx</title>
      </head>
      <body>
            <h1>This file served with NGINX</h1>
      </body>
</html>
```

In `/data/images/` put some images like `example.png` to serve it later using nginx, then Open the 
configuration file & replace its content with:

```nginx
http {
      server {
            listen 8080;

            location / {
                  root /data/www;
            }

            location /images {
                  root /data;
            }
      }
}

events {} # ignore this directive but it required
```

Now you can re/start nginx with

```shell
# in case nginx was shut
nginx
# or see in next section for more detail to reload configuration
nginx -s reload
```

visit [http://localhost:8080/](http://localhost:8080/) for html page or [http://localhost:8080/images/example.png](http://localhost:8080/images/example.png) for the image.
So let explain what all that mean: 

- `http` context(block directive contain other blocks) to serve `http` requests
- `server` context to specify the configuration of virtual server
- `listen` inform nginx the number of port to serve content
- `location` sets configuration depending on a request URI, in our case there is 2 location blocks the 
first one tell nginx to serve all request prefixed with `/` & the secondth to serve all request containing 
`/images` in url for example serve all request of `http://localhost:8080/images/example.png`
- `root` the path where to serve the request.
- `events` required by nginx let ignore it for now.

***Note***:
For matching requests, the URI will be added to the path specified in the `root` directive, that is, to `/data/www`.
If there are several matching location blocks nginx selects the one with the longest prefix.

### Reload new configuration

After editing the configuration file on `/usr/local/etc/nginx/nginx.conf` you can stop & start 
nginx in 2steps or simply reload it using:

```shell
nginx -s reload
```

When reload the configuration nginx verify if the new configuration are valid, if it is then 
the master process start new worker processes & send a signal to old ones to shutdown after 
serving all current request & not accepting new connections. Otherwise the master process 
roll back & keep using the old configuration. For more [control of nginx](https://nginx.org/en/docs/control.html).

### Config a Proxy server

To create a `proxy server` with nginx it realy simple by creating new directory `/data/proxy-server` 
containing html files then edit the config file & add new `server` context to the previous configuration like:

```nginx
server {
      listen 8888;
      root /data/proxy-server;
      location / {}
}
```

that mean, serve a request from port `8888` & files from `/data/proxy-server` then edit the server 
contexts from previous configuration to be:

```nginx
server {
      listen 8080;
      location {
            proxy_pass http://localhost:8888;
      }

      location ~ \.(gif|jpg|png)$ {
            root /data/images;
      }
}
```

if you notice we add new directive `proxy_pass`, this send an instruction to nginx to inform it 
to proxy all requests from `http://localhost:8080` to the new url `http://localhost:8000`, 
in other words if you vist `http://localhost:8080` nginx serve you `http://localhost:8000`,
& second location directive is start with `~` which mean is a regular expression to math one 
of those extensions `gif` or `jpg` or `png` & serve them from `/data/images` so if you visit 
`http://localhost:8080/example.png` will work but not `http://localhost:8000/example.png`;

### Some Handy Directives

Let assume that your config file grows & annoyes you to handle it in one file so you can use separate 
your configuration to several files & include it using `include` directive, which is of type 
simple(not block directive), to put your configuration files following this syntaxe:

```nginx
# here you can write some configuration on main context

# here include external config which contain http directive
include http.conf

# events directive
events {} 
```

Another directive which can be in `main` context is `worker_processes` which accept as a 
parametre a number or `auto` to specify the number of worker processes to use, however those 
number can related to some condition like a number of CPU core, the number of hard disk 
drives that store data, & load pattern.

In some cases, you want to debug your server or see errors *nginx*  fired. Nginx give you 
a possibility to specify a custom path to file who include those errors/debug logs using
`error_log`, which accept 2 parametre the first one is the path to given file, if it dosn't 
exist yet `nginx` will created for you(not the folder just files) & the second one is the type 
of logs to save in that file it one of `debug`, `info`, `notice`, `warn`, `error`, `crit`, `alert`, or 
`emerg`, if you specified `error` the log file will contain `error`, `crit`, `alert`, or `emerg` logs too. 
This directive can be in thos contexts `main`, `http`, `mail`, `stream`, `server`, `location`.

## Caching with NGINX

Caching is a weapon with 2 side, it can be good & bad if you dont know how to exploite it. Caching 
can enhance your application performance & take it to the next level, so in this section will use 
`nginx` to cache some content. To do that its very simple using 3 directives. First, let create a proxy 
server with:

```nginx
http {
	server {
		listen 8888;
		location / {
		      root /Users/mac/Desktop/nginx/proxy-server;
		}
	}

	server {
		listen 8080;
		location / {
                  proxy_pass http://localhost:8888;
		}
	}
}

events {}
```

for now there is nothin new. Second, let user [proxy_cache_path](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_path) to define the path where save data, size of cache, max size of cache so add the following configuration in http context before 2nd server context:

```nginx
#...
proxy_cache_path /data/cache levels=1:2 max_size=1g keys_zone=app_cache_zone_name:10m use_temp_path=off inactive=60m:
# ...
```

this inform nginx to save cache in `/data/cache` or create it if not exist with 2 levels of subfolders
(best practice is to kepp this on `1:2` or less) with the maximum size of 1 Giga, if it exceed nginx will 
remove the least used cached data, then save the cache meta data on the zone `app_cache_zone_name` with 
size of 10 Miga. The content cached with `nginx` created imediatly without passing with temporare folder 
when using `use_temp_path=off` param & refresh those cached content if not reached in 60 minutes. Howeer 
this cache not enabled yet, to do so will add those directive in 2nd `server` context:

```nginx
server {
            listen 8080;
            location / {
                  proxy_cache app_cache_zone_name;
                  proxy_cache_valid 200 10m;
                  proxy_pass http://localhost:8888;
            }
	}
```
those new directives mean to enable the caching zone `app_cache_zone_name` used as param of [proxy_cache](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache) 
& [proxy_cache_valid](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_cache_valid) to inform `nginx` to cache all http requests content served with status of **200** for 10 minutes. Finally 
you can reload the configuration by sending a signal of `HUP` using:

```nginx 
nginx -s reload
```
& just like that you implemented a caching using `nginx` to emprove your app performance.