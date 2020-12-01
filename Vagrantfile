# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/buster64"

  # apt-cache-ng and docker registry cacheing proxy
  config.vm.network "forwarded_port", guest: 3142, host: 3142, host_ip: "0.0.0.0"
  config.vm.network "forwarded_port", guest: 5000, host: 5000, host_ip: "0.0.0.0"

  # disable /vagrant sync folder
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider "libvirt" do |l|
    l.cpus = 2
    l.memory = "512"
    l.qemu_use_session = false
    # Allow unmap in guest
    l.disk_bus = "scsi"
    # Restart on host reboot
    #l.autostart = true
  end

  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    apt-get update; apt-get install -y apt-cacher-ng auto-apt-proxy
    apt-get update; apt-get install -y zram-tools unattended-upgrades docker.io

    #grep -q kubernetes /etc/apt-cacher-ng/acng.conf || echo "Remap-k8: apt.kubernetes.io
    #https://apt.kubernetes.io" >> /etc/apt-cacher-ng/acng.conf
    systemctl restart apt-cacher-ng
    apt-get update

    (( $(docker ps -q --filter "name=registry" | wc -l) == 1 )) || docker run \
      --name registry \
      --restart unless-stopped \
      -d \
      -e REGISTRY_PROXY_REMOTEURL="https://registry-1.docker.io" \
      -p 5000:5000 \
      -v /srv/registry:/var/lib/registry \
      registry:2

    systemctl enable fstrim.timer --now
  SHELL
end

