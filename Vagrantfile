# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/buster64"

  config.vm.network "forwarded_port", guest: 3142, host: 3142, host_ip: "0.0.0.0"
  config.vm.define "buster"

  config.vm.provider "virtualbox" do |vb|
    vb.memory = "256"
    vb.cpus = 1
    vb.linked_clone = true
    vb.default_nic_type = "virtio"
  end

  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    apt-get update
    apt-get install -y apt-cacher-ng auto-apt-proxy
    apt-get update
    apt-get install -y zram-tools unattended-upgrades
    grep -q kubernetes /etc/apt-cacher-ng/acng.conf || echo "Remap-k8: apt.kubernetes.io ; https://apt.kubernetes.io" >> /etc/apt-cacher-ng/acng.conf
    systemctl restart apt-cacher-ng
    apt-get update
  SHELL
end

