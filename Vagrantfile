# -*- mode: ruby -*-
# vi: set ft=ruby :

private_ip_base = "192.168.56.1"

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.vm.provider "virtualbox" do |v|
    v.memory = 512
    v.cpus = 1
  end
  
  (1..2).each do |i|
    config.vm.define "mysql#{i}" do |node|
      node.vm.hostname = "mysql#{i}"
      node.vm.network "private_network", ip: "#{private_ip_base}#{i}", adapter: 3
    end
  end

  (3..4).each do |i|
    config.vm.define "wp#{i}" do |node|
      node.vm.hostname = "wp#{i}"
      node.vm.network "private_network", ip: "#{private_ip_base}#{i}", adapter: 3
      node.vm.provider "virtualbox" do |vb|
        unless File.exist?(".drbd#{i}.vdi")
          vb.customize ['createhd', '--filename', ".drbd#{i}.vdi", '--variant', 'Fixed', '--size', '1024']
          needsController = true
        end
        vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata"]
        vb.customize ['storageattach', :id, '--storagectl', 'SATA', '--port', 1, '--device', 0, '--type', 'hdd', '--medium', ".drbd#{i}.vdi"]
      end 
    end
  end  

  config.vm.define "borg" do |borg|
    borg.vm.hostname = "borg"
    borg.vm.network "private_network", ip: "#{private_ip_base}5", adapter: 3
  end

  config.vm.define "nginx" do |nginx|
    nginx.vm.hostname = "nginx"
    nginx.vm.network "private_network", ip: "#{private_ip_base}0", adapter: 3
    nginx.vm.network "forwarded_port", guest: 80, host: 80
    nginx.vm.network "forwarded_port", guest: 443, host: 443

    nginx.vm.provision "ansible" do |ansible|
      ansible.playbook = "main.yml"
      ansible.inventory_path = "hosts"
      ansible.limit = "all"
      ansible.verbose = "v"
      #ansible.tags = "borg,client"
    end
  end  
end

