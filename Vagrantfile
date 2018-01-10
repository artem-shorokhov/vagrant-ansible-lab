Vagrant.configure("2") do |config|
  
  PUPPETEER_BOX  = "bento/ubuntu-16.04"
  PUPPETEER      = { :hostname => "ansible" }
  # PUPPETEER      = { :hostname => "ansible", :ip => "192.168.68.10" }
  PUPPETS_BOX    = "bento/centos-7.4"
  PUPPETS        = [
    { :hostname => "puppet" }
    # { :hostname => "puppet", :ip => "192.168.68.11" }
  ]
  
  # Set up target server(s).
  PUPPETS.each do |puppet|
    config.vm.define puppet[:hostname] do |host|
      host.vm.box = PUPPETS_BOX
      host.vm.hostname = puppet[:hostname]
      host.vm.network "public_network"
      # host.vm.network "private_network", ip: puppet[:ip]
    end
  end
  
  # Set up ansible server.
  config.vm.define PUPPETEER[:hostname] do |host|
  
    host.vm.box = PUPPETEER_BOX
    host.vm.hostname = PUPPETEER[:hostname]
    host.vm.network "public_network"
    # host.vm.network "private_network", ip: PUPPETEER[:ip]

    # Generate SSH key pair & upload to each target server.
    host.vm.provision "shell", inline: "ssh-keygen -f /home/vagrant/.ssh/id_rsa -N '' -t rsa"
    host.vm.provision "shell", inline: "apt-get update && apt-get install sshpass"
    
    PUPPETS.each do |puppet|
      host.vm.provision "shell" do |shell|
        shell.inline = "sshpass -p vagrant ssh-copy-id -i /home/vagrant/.ssh/id_rsa -o StrictHostKeyChecking=no vagrant@$1"
        shell.args   = puppet[:hostname]
      end
    end
    
    host.vm.provision "shell", inline: "cp ~/.ssh/known_hosts /home/vagrant/.ssh/known_hosts"
    host.vm.provision "shell", inline: "chown vagrant:vagrant /home/vagrant/.ssh/*"
  
    # Install Ansible.
    host.vm.provision "shell",
      inline: <<-SCRIPT
        apt-get update
        apt-get install software-properties-common
        apt-add-repository -y ppa:ansible/ansible
        apt-get update
        apt-get install -y ansible
      SCRIPT
    
    # Configure Ansible-related stuff.
    host.vm.provision "shell",
      inline: <<-SCRIPT
        mv /etc/ansible/hosts /etc/ansible/hosts.default
        ln -s /vagrant/hosts /etc/ansible/hosts
      SCRIPT
    
  end
  
end
