# -*- mode: ruby -*-
# vi: set ft=ruby :

server_ip = "192.168.33.10"
agents = { "agent1" => "192.168.33.11"}
#agents = { "agent1" => "192.168.33.11",
#           "agent2" => "192.168.33.12",
#           "agent3" => "192.168.33.13" }

# Extra parameters in INSTALL_K3S_EXEC variable because of
# K3s picking up the wrong interface when starting server and agent
# https://github.com/alexellis/k3sup/issues/306

server_script = <<-SHELL
    sudo -i
    apk add curl
    export INSTALL_K3S_EXEC="--bind-address=#{server_ip} --node-external-ip=#{server_ip} --flannel-iface=eth0"
    curl -sfL https://get.k3s.io | sh -
    echo "Sleeping for 5 seconds to wait for k3s to start"
    sleep 5
    cp /var/lib/rancher/k3s/server/token /vagrant_shared
    cp /etc/rancher/k3s/k3s.yaml /vagrant_shared
    SHELL

agent_script = <<-SHELL
    sudo -i
    apk add curl
    export K3S_TOKEN_FILE=/vagrant_shared/token
    export K3S_URL=https://#{server_ip}:6443
    export INSTALL_K3S_EXEC="--flannel-iface=eth1"
    curl -sfL https://get.k3s.io | sh -
    SHELL

Vagrant.configure("2") do |config|
  config.vm.box = "generic/alpine314"

  config.vm.define "server", primary: true do |server|
    server.vm.network "private_network", ip: server_ip
    server.vm.synced_folder "./shared", "/vagrant_shared"
    server.vm.hostname = "server"
    server.vm.enable_virtualization_extensions = true
    server.vm.provider "hyperv" do |vb|
      vb.memory = "2048"
      vb.cpus = "2"
    end
    server.vm.provision "shell", inline: server_script
  end

  agents.each do |agent_name, agent_ip|
    config.vm.define agent_name do |agent|
      agent.vm.network "private_network", ip: agent_ip
      agent.vm.synced_folder "./shared", "/vagrant_shared"
      agent.vm.enable_virtualization_extensions = true
      agent.vm.hostname = agent_name
      agent.vm.provider "hyperv" do |vb|
        vb.memory = "2048"
        vb.cpus = "2"
      end
      agent.vm.provision "shell", inline: agent_script
    end
  end
end