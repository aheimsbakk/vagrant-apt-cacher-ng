# Debian Buster with `apt-cacher-ng`

This `Vagrantfile` creates a small Debian Buster VM with `apt-cacher-ng` installed. `apt-cacher-ng` is a caching proxy for Debian packages, and may speed up your development.

* 192M
* `apt-cacher-ng` installed
* port `3142` is forwarded to host `0.0.0.0`

## Usage

On other Vagrant VMs, add the following to utilize `apt-cacher-ng`. Last line is to get a fully upgraded VM.

```
config.vm.provision "shell", inline: <<-SHELL
  apt-get update; apt-get install -y auto-apt-proxy; apt-get update
  DEBIAN_FRONTEND=noninteractive apt-get dist-upgrade -y
SHELL
```

###### vim: spell spelllang=en ts=2 st=2 et ai:

