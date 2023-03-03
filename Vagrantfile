boxes = [
  { :name => "backup",
    :ip => "192.168.56.10",
    :ram => 1024,
    :hdd_name => "backup.vdi",
    :hdd_size => 4096
  },
  { :name => "client",
    :ip => "192.168.56.15",
    :ram => 1024
  }
]

Vagrant.configure("2") do |config|
  boxes.each do |machine|
    config.vm.define machine[:name] do |config|
      config.vm.box = "centos/7"
      config.vm.box_version = "2004.01"
      config.vm.hostname = machine[:name]
      config.vm.network "private_network", ip: machine[:ip]
      config.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--memory", machine[:ram]]  
        if (!machine[:hdd_name].nil?)
          unless File.exist?(machine[:hdd_name])
          vb.customize ["createmedium", "--filename", machine[:hdd_name], "--size", machine[:hdd_size]]
          needsController =  true
          end
          vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata"]
          vb.customize ["storageattach", :id, "--storagectl", "SATA", "--port", 1, "--device", 0, "--type", "hdd", "--medium", machine[:hdd_name]]
        
        end
      end
        
      if machine[:name] == boxes.last[:name]
        config.vm.provision "ansible" do |ansible|
          ansible.playbook = "ansible/borg.yml"
          ansible.limit = "all"
        end  
      end
    end
  end
end