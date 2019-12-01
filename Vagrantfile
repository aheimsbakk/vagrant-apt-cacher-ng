# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/buster64"

  config.vm.network "forwarded_port", guest: 3142, host: 3142, host_ip: "0.0.0.0"
  config.vm.define "buster"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "256"
    vb.cpus = 1
  end

  config.vm.provision "shell", inline: <<-SHELL
    apt-get update
    export DEBIAN_FRONTEND=noninteractive
    apt-get install -y apt-cacher-ng auto-apt-proxy
    grep -q kubernetes /etc/apt-cacher-ng/acng.conf || echo "Remap-k8: apt.kubernetes.io ; https://apt.kubernetes.io" >> /etc/apt-cacher-ng/acng.conf
    systemctl restart apt-cacher-ng
    apt-get update
    apt-get install -y unattended-upgrades
    apt-get dist-upgrade -y
  SHELL
end

