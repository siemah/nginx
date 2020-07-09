# NGINX

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
file is named nginx.conf and placed in the directory `/usr/local/nginx/conf`, `/etc/nginx`, 
or `/usr/local/etc/nginx`. 



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
