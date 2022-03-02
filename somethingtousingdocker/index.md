# something to using docker


## set proxy

It's annoying that you can't download third-party libraries from github because of network when using linux os docker image to develop. So using proxy in docker is necessary.

[This is the official website to use proxy in docker](https://docs.docker.com/config/daemon/systemd/)

Below is one of the ways if you have root' permission

```bash
sudo mkdir -p /etc/systemd/system/docker.service.d
sudo touch /etc/systemd/system/docker.service.d/proxy.conf
```
add the below content to `proxy.conf`
```bash
sudo touch /etc/systemd/system/docker.service.d/proxy.conf
```
then restart docker
```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
```
**notice:** This is not to say that docker container will connect to the port in your physics machine. You still need to install your vpn client in each of your containers.

## install libraries

It's common to find that a linux os container lacks of libraries when linking because linux docker image is minimal. So how to install libraries when getting an error like `cannot find -lcrc32c` if you don't know the library's name? In ubuntu, you can install `libcrc*`.`l` means `lib` and you can use `tab` to autocompletion the library's name, and then you can find the library you need is called `libcrcutil-dev`.
