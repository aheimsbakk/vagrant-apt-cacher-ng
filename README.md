# Debian Buster with `apt-cacher-ng` and docker proxy

This `Vagrantfile` creates a small Debian Buster VM with `apt-cacher-ng` and a docker caching proxy installed. `apt-cacher-ng` is a caching proxy for Debian packages, and may speed up your development.

VM is configured with `368M` memory. Port `3142` and `5000` is forwarded to hosts `0.0.0.0`

Packages installed.

* `apt-cacher-ng`
* `auto-apt-proxy`
* `docker.io`
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

Use the following to enable k8 repos. Change `kubernetes-xenial` to your distro.

```ruby
config.vm.provision "shell", inline: <<-SHELL
  echo "deb http://apt.kubernetes.io kubernetes-xenial main" > /etc/apt/sources.list.d/kubernetes.list
  apt-get update
SHELL
```



###### vim: spell spelllang=en ts=2 st=2 et ai:

