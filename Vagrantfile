# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "generic/ubuntu2004"

  config.vm.provider :libvirt do |v|
    v.qemu_use_session = false 
  end

  # Define VMs with static private IP addresses.
  boxes = [
    { :name => "controller-1", :ip => "192.168.1.21" },
    { :name => "worker-1", :ip => "192.168.1.31" },
    { :name => "worker-2", :ip => "192.168.1.32"},
  ]

  # Provision each of the VMs.
  boxes.each do |opts|
    config.vm.define opts[:name] do |config|
      config.vm.hostname = opts[:name]
      config.vm.network :private_network, ip: opts[:ip]
      if opts[:name] == "worker-1" || opts[:name] == "worker-2"
        config.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "2048"]
          vb.customize ["modifyvm", :id, "--cpus", "2"]
        end
      end
      if opts[:name] == "controller-1" || opts[:name] == "controller-2" || opts[:name] == "controller-3"
        config.vm.provider :virtualbox do |vb|
          vb.customize ["modifyvm", :id, "--memory", "2048"]
          vb.customize ["modifyvm", :id, "--cpus", "2"]
        end
      end
      # Provision all the VMs in parallel using Ansible after last VM is up.
      if opts[:name] == "worker-2"
        config.vm.provision "ansible" do |ansible|
          ansible.compatibility_mode = "2.0"
          ansible.playbook = "main.yml"
          ansible.limit = "all"
          ansible.become = true
          ansible.groups = {
            "kubernetes" => ["worker-1", "worker-2",
                             "controller-1", "controller-2", "controller-3"],
            "controller" => ["controller-1", "controller-2", "controller-3"],
            "etcd" => ["etcd-1", "etcd-2", "etcd-3"],
            "worker" => ["worker-1", "worker-2"],
            "worker:vars" => {
              kubernetes_role: "worker",
            }
          }
        end
      end
    end
  end

end
