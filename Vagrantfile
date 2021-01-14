# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.box_check_update = false

  # Use this if you don't have a bridge. apt-cache-ng and docker registry cacheing proxy.
  #config.vm.network "forwarded_port", guest: 3142, host: 3142, host_ip: "0.0.0.0"
  #config.vm.network "forwarded_port", guest: 5000, host: 5000, host_ip: "0.0.0.0"

  # use this if you want a public network, change to you own brigde interface or remove for interactive configuration.
  config.vm.network "public_network", bridge: "bridge0", ip: "192.168.1.11"

  config.vm.synced_folder ".", "/vagrant", disabled: true

  config.vm.define "proxy" do |config|
    config.vm.hostname = "proxy"
  end

  config.vm.provider "virtualbox" do |vb|
    vb.cpus = 2
    vb.memory = "512"
    vb.linked_clone = true
  end

  config.vm.provision "apt cache", type: "shell", inline: <<-SHELL
    export DEBIAN_FRONTEND=noninteractive
    apt-get update
    apt-get install -y apt-cacher-ng auto-apt-proxy zram-tools unattended-upgrades

    grep -q kubernetes /etc/apt-cacher-ng/acng.conf || echo "Remap-k8: apt.kubernetes.io https://apt.kubernetes.io" >> /etc/apt-cacher-ng/acng.conf
    systemctl restart apt-cacher-ng
  SHELL

  config.vm.provision "containers", type: "docker" do |p|
    p.run "registry",
      image: "docker.io/registry:2",
      args: "-e REGISTRY_PROXY_REMOTEURL=https://registry-1.docker.io \
             -e REGISTRY_STORAGE_DELETE_ENABLED=true \
             -p 5000:5000 \
             -v /srv/registry:/var/lib/registry"
    p.run "watchtower",
      image: "docker.io/containrrr/watchtower:latest",
      cmd: "-i 86400",
      args: "-v /var/run/docker.sock:/var/run/docker.sock"
  end

  config.vm.provision "registry cleanup", type: "shell", inline: <<-SHELL
    cat <<EOF > /etc/cron.daily/registry-cleanup
#!/bin/bash
if [ \\$(docker ps -q --filter "name=registry" | wc -l)  -eq 1 ]; then
  docker exec -t registry bin/registry garbage-collect -m /etc/docker/registry/config.yml | systemd-cat -t CRON_REGISTRY
  docker restart registry
fi
EOF
    chmod +x /etc/cron.daily/registry-cleanup
  SHELL

  config.vm.provision "enable firewall", type: "shell", inline: <<-SHELL
    ufw allow 22/tcp
    ufw allow 5000/tcp
    ufw allow 3142/tcp
    yes | ufw enable
  SHELL
end

