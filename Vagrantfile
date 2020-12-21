# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "debian/buster64"

  ## Use this if you don't have a bridge. apt-cache-ng and docker registry cacheing proxy.
  #config.vm.network "forwarded_port", guest: 3142, host: 3142, host_ip: "0.0.0.0"
  #config.vm.network "forwarded_port", guest: 5000, host: 5000, host_ip: "0.0.0.0"

  # Use this if you have a bridge0. Give vagrant an "public" ip on computer bridge0.
  config.vm.network "public_network", dev: "bridge0", mode: "bridge", type: "bridge", ip: "192.168.1.11"

  # disable /vagrant sync folder
  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.provider "libvirt" do |l|
    l.cpus = 2
    l.memory = "512"
    l.qemu_use_session = false
    # Allow unmap in guest
    l.disk_bus = "scsi"
    #l.disk_driver discard: "unmap", detect_zeroes: "unmap"
    # Restart on host reboot
    l.autostart = true
  end

  config.vm.provision "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    apt-get update; apt-get install -y apt-cacher-ng auto-apt-proxy
    apt-get update; apt-get install -y zram-tools unattended-upgrades docker.io ufw

    ufw allow 22/tcp
    ufw allow 5000/tcp
    ufw allow 3142/tcp
    yes | ufw enable

    grep -q kubernetes /etc/apt-cacher-ng/acng.conf || echo "Remap-k8: apt.kubernetes.io https://apt.kubernetes.io" >> /etc/apt-cacher-ng/acng.conf
    systemctl restart apt-cacher-ng
    apt-get update

    (( $(docker ps -q --filter "name=registry" | wc -l) == 1 )) || docker run \
      --name registry \
      --restart unless-stopped \
      -d \
      -e REGISTRY_PROXY_REMOTEURL="https://registry-1.docker.io" \
      -e REGISTRY_STORAGE_DELETE_ENABLED="true" \
      -p 5000:5000 \
      -v /srv/registry:/var/lib/registry \
      docker.io/registry:2

    (( $(docker ps -q --filter "name=watchtower" | wc -l) == 1 )) || docker run \
      --name watchtower \
      --restart unless-stopped \
      -v /var/run/docker.sock:/var/run/docker.sock \
      -d \
      docker.io/containrrr/watchtower:latest -i 3600

    cat <<EOF > /etc/cron.hourly/registry-cleanup
#!/bin/bash
[ \\$(docker ps -q --filter "name=registry" | wc -l)  -eq 1 ] && docker exec -t registry bin/registry garbage-collect -m /etc/docker/registry/config.yml | systemd-cat -t CRON_REGISTRY
EOF
    chmod +x /etc/cron.hourly/registry-cleanup
    systemctl enable fstrim.timer --now
  SHELL
end

