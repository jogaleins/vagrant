# -*- mode: ruby -*-
# vi: set ft=ruby :




server_ip = "192.168.56.10"
#agents = { "agent1" => "192.168.56.11"}
gateway_ip = "192.168.56.1"
agents = { "agent1" => "192.168.56.11",
           "agent2" => "192.168.56.12"}
 #          "agent3" => "192.168.33.13" }
pub_key = "ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDoru+s4pjmh9pCaA/FtPT4BxOblHRXwOOkBbhoK/gJIrPdPxx+aneGNhlpc+jqjF75X2Sa7dwZAzeyNdO9MdtlqowSyZj6xVHP6nLUMfA8tppgODGv231m8NBnIrQvUlMrzLgwZQHM8P8s6LjA3bLCJGG3t/9xmfxZsCNPqFjGHFLJhyvA3BrGEI1PQnBVCzkpkVxbUEXrFkjKgCt5TkL0eV3mj+MGMx/gFOHZxHKMVuEnpdSSH2Z8eXWL03cavRa+B75mXcSmdJOlYP90eUg1t23O+Ss3/tooAgdCkMUzsOn/eBpDVZNUhFzMOY/QJ1tPq0mGPsY29cWIc+ZYAnMAdZbxxNBcklF9NxxEoxfGCu8FLB4CQrallVjTRCR15/zcFqvizAUsHDfWeLUpyRG1zHUxHMS+is2cE+5FrXBzhvQppkE98a6t/vJha7T819JhwXBOleHRft0KNRdxcGFim3+cu323f+ufc6QcvZzAjwWF/JyUL78lZZ6EsCg7Kbs= ap@ap"
# Extra parameters in INSTALL_K3S_EXEC variable because of
# K3s picking up the wrong interface when starting server and agent
# https://github.com/alexellis/k3sup/issues/306

server_static_ip_script = <<-SHELL
    sudo -i
    apk add curl
    echo "auto lo" > /etc/network/interfaces
    echo "iface lo inet loopback" >> /etc/network/interfaces
    echo " " >> /etc/network/interfaces
    echo "auto eth0" >> /etc/network/interfaces
    echo "iface eth0 inet static" >> /etc/network/interfaces
    echo "  address #{server_ip}/24" >> /etc/network/interfaces
    echo "  gateway #{gateway_ip}" >> /etc/network/interfaces
    echo "  hostname server-#{server_ip}" >> /etc/network/interfaces
    #/etc/init.d/networking restart
    #setup-interface -i /etc/network/interfaces
    SHELL

    server_init_k3s_script = <<-SHELL
    sudo -i
    sleep 5
    #curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server --cluster-init --flannel-iface=eth1 --disable traefik --disable servicelb
    curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server --cluster-init --flannel-iface=eth1 --disable servicelb
    echo "Sleeping for 60 seconds to wait for k3s to start"
    sleep 60
    cp /var/lib/rancher/k3s/server/token /vagrant_shared
    cp /etc/rancher/k3s/k3s.yaml /vagrant_shared
    apk update
    apk add ansible
	echo #{pub_key} >> /home/vagrant/.ssh/authorized_keys
    SHELL

Vagrant.configure("2") do |config|
  config.vm.box = "generic/alpine314"
 
  config.vm.define "server", primary: true do |server|
    #server.vm.network "private_network",bridge: "vmbr1", ip: server_ip
    server.vm.network :private_network, ip: server_ip
    server.vm.synced_folder "./shared", "/vagrant_shared",  type: "smb",
      smb_password: "bo0m", smb_username: "adp",
      mount_options: ["username=adp","password=bo0m"]
    #server.vm.synced_folder "./shared", "/vagrant_shared", disabled: true

    server.vm.hostname = "server"
    
    server.vm.provider "virtualbox" do |vb|
      #vb.enable_virtualization_extensions = true
      vb.memory = "2048"
      vb.cpus = "2"
    end
    #server.vm.provision "shell", inline: server_static_ip_script
    #server.vm.provision :unix_reboot
    server.vm.provision "shell", inline: server_init_k3s_script
  end

  agents.each do |agent_name, agent_ip|
    config.vm.define agent_name do |agent|
      agent.vm.network :private_network, ip: agent_ip
      agent.vm.synced_folder "./shared", "/vagrant_shared", type: "smb",
        smb_password: "bo0m", smb_username: "adp",
        mount_options: ["username=adp","password=bo0m"]
      agent.vm.hostname = agent_name
      agent.vm.provider "virtualbox" do |vb|
      #  vb.enable_virtualization_extensions = true
        vb.memory = "2048"
        vb.cpus = "2"
      end
    agent_static_ip_script = <<-SHELL
    sudo -i
    echo "auto lo" > /etc/network/interfaces
    echo "iface lo inet loopback" >> /etc/network/interfaces
    echo " " >> /etc/network/interfaces
    echo "auto eth0" >> /etc/network/interfaces
    echo "iface eth0 inet static" >> /etc/network/interfaces
    echo "  address #{agent_ip}/24" >> /etc/network/interfaces
    echo "  gateway #{gateway_ip}" >> /etc/network/interfaces
    echo "  hostname server-#{agent_ip}" >> /etc/network/interfaces
    #/etc/init.d/networking restart
    #setup-interface -i /etc/network/interfaces

	
    SHELL

    agent_k3s_init_script = <<-SHELL
    sleep 5
    apk add curl
    #curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server --server https://#{server_ip}:6443 --flannel-iface=eth1 --disable traefik --disable servicelb
    curl -sfL https://get.k3s.io | K3S_TOKEN=SECRET sh -s - server --server https://#{server_ip}:6443 --flannel-iface=eth1 --disable servicelb
    apk update
    apk add ansible
	echo #{pub_key} >> /home/vagrant/.ssh/authorized_keys
    SHELL

      #agent.vm.provision "shell", inline: agent_static_ip_script
      #agent.vm.provision :unix_reboot
      agent.vm.provision "shell", inline: agent_k3s_init_script
    end
  end
end

#system('vagrant ssh server -c \'sudo cat /etc/rancher/k3s/k3s.yaml\' > k3s.yaml')