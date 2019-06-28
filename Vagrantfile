Vagrant.configure("2") do |config|
  config.vm.provider "virtualbox" do |v|
    v.memory = 8192
    v.cpus = 4
    v.name = "devops_vm"
    v.customize [ "guestproperty", "set", :id, "/VirtualBox/GuestAdd/VBoxService/--timesync-set-threshold", 1000 ]
  end

  config.vm.box = "bento/ubuntu-19.04"
  config.disksize.size = '50GB'

  # enable remote ssh from visual studio code
  config.ssh.insert_key = false # 1
  config.ssh.private_key_path = ['~/.vagrant.d/insecure_private_key', '~/.ssh/id_rsa'] # 2
  config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/authorized_keys" # 3

  # 4
  config.vm.provision "shell", inline: <<-EOC
    sudo sed -i -e "\\#PasswordAuthentication yes# s#PasswordAuthentication yes#PasswordAuthentication no#g" /etc/ssh/sshd_config
    sudo systemctl restart sshd.service
    echo "finished"
  EOC

  config.vm.network "public_network"
  
  config.vm.network "forwarded_port", guest: 80, host: 8080
  config.vm.network "forwarded_port", guest: 443, host: 8443

  config.vm.provision "ansible_local" do |ansible|
    ansible.playbook = "ansible-install.yml"
    #ansible.verbose  = true
    ansible.become   = true
  end
  
end
