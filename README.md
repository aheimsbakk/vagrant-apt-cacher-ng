# Debian Buster with `apt-cacher-ng`

This `Vagrantfile` creates a small Debian Buster VM with `apt-cacher-ng` installed. `apt-cacher-ng` is a caching proxy for Debian packages, and may speed up your development.

VM is configured with `256MB` memory and port `3142` is forwarded to hosts `0.0.0.0`

Packages installed.

* `apt-cacher-ng`
* `auto-apt-proxy`
* `unattended-upgrades`
* `zram-tools`

In addition `apt.kubernetes.io` is added to apt-cacher-ng configuration.

## Usage

On other Vagrant VMs, add the following to utilize `apt-cacher-ng`. Last line is to get a fully upgraded VM.

```ruby
config.vm.provision "shell", inline: <<-SHELL
  export DEBIAN_FRONTEND=noninteractive
  apt-get update; apt-get install -y auto-apt-proxy; apt-get update
  apt-get dist-upgrade -y
SHELL
```

### Kubernetes packages

Update `sources.list` with the following line. Change `kubernetes-xenial` to your distro.

```ruby
config.vm.provision "shell", inline: <<-SHELL
  echo "deb http://apt.kubernetes.io kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
  apt-get update
SHELL
```



###### vim: spell spelllang=en ts=2 st=2 et ai:

