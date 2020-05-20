Vagrant.configure("2") do |config|
    # List of servers
    servers=[
      {
        :hostname => "ubuntu-server",
        :box => "ubuntu/bionic64",
        :ip => "192.168.50.1",
        :ssh_port => '2211',
        :nginx_port => '80',
        :nginx_https_port => '443'
      }
    ]
  
    servers.each do |machine|
      
      config.vm.define machine[:hostname] do |node|
        node.vm.box = machine[:box]
        node.vm.hostname = machine[:hostname]
      
        node.vm.network :private_network, ip: machine[:ip]
        node.vm.network "forwarded_port", guest: 22, host: machine[:ssh_port], id: "ssh"
        node.vm.network "forwarded_port", guest: 80, host: machine[:nginx_port], id: "http"
        node.vm.network "forwarded_port", guest: 443, host: machine[:nginx_https_port], id: "https"
        node.vm.provider :virtualbox do |v|
          v.customize ["modifyvm", :id, "--memory", 2048]
          v.customize ["modifyvm", :id, "--name", machine[:hostname]]
        end
      end
  
    end
  
    # Inject your public SSH key
    id_rsa_key_pub = File.read(File.join(Dir.home, ".ssh", "id_rsa.pub"))
  
    config.vm.provision :shell,
          :inline => "echo 'appending SSH public key to ~vagrant/.ssh/authorized_keys' && echo '#{id_rsa_key_pub }' >> /home/vagrant/.ssh/authorized_keys && chmod 600 /home/vagrant/.ssh/authorized_keys"
  
    config.ssh.insert_key = false
  
    #config.vm.provision "ansible" do |ansible|
      #ansible.playbook = "site.yml"
    #end
  end