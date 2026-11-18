Vagrant.configure("2") do |config|
  config.ssh.insert_key = false

  box="ubuntu/jammy64"
  base_network="192.168.56."

  config.vm.define :nodo1 do |nodo1|
    nodo1.vm.box=box
    nodo1.vm.hostname="nodo1"
    nodo1.vm.network :private_network,ip:"#{base_network}101"

    nodo1.vm.provider "virtualbox" do |vb|
      vb.name="vagrant-nodo1"
      vb.memory=1024
      vb.cpus=1
    end
  end

  config.vm.define :nodo2 do |nodo2|
    nodo2.vm.box=box
    nodo2.vm.hostname="nodo2"
    nodo2.vm.network :private_network,ip:"#{base_network}102"
    
    nodo2.vm.provider "virtualbox" do |vb|
      vb.name="vagrant-nodo2"
      vb.memory=1024
      vb.cpus=1
    end
  end

  config.vm.define :balanceador do |balanceador|
    balanceador.vm.box=box
    balanceador.vm.hostname="balanceador"
    balanceador.vm.network :private_network,ip:"#{base_network}100"
    balanceador.vm.network :forwarded_port,guest:80,host:8080

    balanceador.vm.provider "virtualbox" do |vb|
      vb.name="vagrant-balanceador"
      vb.memory=512
      vb.cpus=1
    end

    
    balanceador.vm.provision "ansible" do |ansible|
      ansible.playbook="ansible/playbook.yml"
      ansible.limit="all"
      ansible.groups={
        "web" => ["nodo1","nodo2"],
        "balanceador" => ["balanceador"]
      }
    end
  end
end
