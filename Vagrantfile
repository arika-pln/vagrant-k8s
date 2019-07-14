#
#  https://github.com/oracle/vagrant-boxes/tree/master/OracleDatabase/11.2.0.2
#  https://github.com/oracle/vagrant-boxes/tree/master/Kubernetes
#

# instalacja plugin'a vagrant-proxyconf
unless Vagrant.has_plugin?("vagrant-proxyconf")
    puts 'Installing vagrant-proxyconf Plugin ... '
    system('vagrant plugin install vagrant-proxyconf')
end

# instalacja plugin'a vagrant-hosts             
unless Vagrant.has_plugin?("vagrant-hosts")     
  puts 'Installing vagrant-hosts Plugin ... '   
  system('vagrant plugin install vagrant-hosts')
end                                             

Vagrant.configure("2") do |config|
  #config.vm.box = "hashicorp/precise32" # 12.04 LTS   
  #config.vm.box = "ubuntu/xenial64"  # v. 16.04 LTS   
  config.vm.box = "ubuntu/bionic64" # 18.04 LTS        
  config.vm.box_check_update = false                   

  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 2
    #v.gui = true
  end

  config.vm.provider "hyperv" do |h|
    h.memory = 1024
    h.cpus = 2
  end

  config.vm.provider "vmware_desktop" do |v|
    v.customize ["modifyvm", :id, "--cpuexecutioncap", "90"]
    v.vmx["memsize"] = "1024"
    v.vmx["numvcpus"] = "2"
  end
  


  if Vagrant.has_plugin?("vagrant-proxyconf")
    config.proxy.http     = "http://192.168.1.10:3128/"
    config.proxy.https    = "http://192.168.1.10:3128/"
    config.proxy.no_proxy = "localhost,127.0.0.1,.example.com"
  end

  config.ssh.insert_key = false

  #VM Edge
  config.vm.define "edge", primary: true do |edge|
    edge.vm.hostname = "edge.k8s"
    edge.vm.network :private_network, ip: "192.168.6.100"
    if Vagrant.has_plugin?("vagrant-hosts")            
      edge.vm.provision :hosts, :sync_hosts => true    
    end                                                

    edge.vm.provision "shell", inline: <<-SHELL
      apt update
      apt install -y software-properties-common
      add-apt-repository ppa:ansible/ansible
      apt update
      apt install -y ansible 
      ansible --version
      # -> ansible 2.8.2, python 2.7.15+
    SHELL

    edge.vm.provision :shell, :inline =>"                       
      ssh-keygen -t rsa -N '' -f .ssh/id_rsa                    
                                                            
      echo 'Host edge master* node* 192.168.*.*' >> .ssh/config 
      echo '  StrictHostKeyChecking no' >> .ssh/config          
      echo '  UserKnownHostsFile /dev/null' >> .ssh/config      
                                                            
      cat .ssh/id_rsa.pub >> .ssh/authorized_keys               
                                                            
      cp -r .ssh /vagrant/                                      
    ", privileged: false                                        
  end

  #VM Master
  config.vm.define "master" do |master|
    master.vm.hostname = "master.k8s"
    master.vm.network :private_network, ip: "192.168.6.101"
    if Vagrant.has_plugin?("vagrant-hosts")
      master.vm.provision :hosts, :sync_hosts => true
    end

    master.vm.provision "shell", inline: <<-SHELL
    SHELL

    master.vm.provision :shell, :inline =>"
      echo 'Copying edge-vm public SSH Keys to the VM'
      cp /vagrant/.ssh/* .ssh/
      chmod -R 600 .ssh/authorized_keys
      chmod -R 600 .ssh/config
      chmod -R 600 .ssh/id_rsa
      chmod -R 644 .ssh/id_rsa.pub
    ", privileged: false
  end
  


=begin
  #VM Node
  config.vm.define "node" do |node|
    node.vm.hostname = "node.k8s"
    node.vm.network :private_network, ip: "192.168.6.102"

    node.vm.provision "ansible" do |ansible|
      ansible.verbose = 'v'
      ansible.playbook = "scripts/test-ansible.yml"
    end  
  end
=end

end