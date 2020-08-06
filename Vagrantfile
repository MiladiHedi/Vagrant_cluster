# -*- mode: ruby -*-
# vi: set ft=ruby :

unless Vagrant.has_plugin?("vagrant-vbguest")
  system("vagrant plugin install vagrant-vbguest")
  puts "Dependencies installed, please try the command again."
  exit
end
unless Vagrant.has_plugin?("vagrant-hostmanager")
  system("vagrant plugin install vagrant-hostmanager")
  puts "Dependencies installed, please try the command again."
  exit
end

$install_ansible= <<SCRIPT
  if ! [ -x "$(command -v ansible)" ]; then
    
      echo "Ansible could not be found, we install it"
      apt-get update
      #apt-get install linux-image-amd64 linux-headers-amd64

      dpkg -s ansible &> /dev/null
    
      echo 'deb http://http.debian.net/debian stretch-backports main' > /etc/apt/sources.list.d/backports.list
      sudo apt update
      sudo apt -t stretch-backports install -y ansible
  fi

SCRIPT

$fix_ssh_permissions= <<-SCRIPT
  chmod 700 /home/vagrant/.ssh/
  chmod 644 /home/vagrant/.ssh/authorized_keys
  chmod 600 /home/vagrant/.ssh/id_rsa
  chmod 644 /home/vagrant/.ssh/id_rsa.pub
SCRIPT

$rewrite_hosts= <<-SCRIPT
  hostIp=$1

  echo "127.0.0.1       localhost" > /etc/hosts
  #echo "127.0.1.1       stretch.localdomain     stretch" >> /etc/hosts
  echo "# The following lines are desirable for IPv6 capable hosts" >> /etc/hosts
  echo "::1     localhost ip6-localhost ip6-loopback" >> /etc/hosts
  echo "ff02::1 ip6-allnodes" >> /etc/hosts
  echo "ff02::2 ip6-allrouters" >> /etc/hosts
  
  echo "192.168.100.10 master-1" >> /etc/hosts
  echo "192.168.101.10 node-1" >> /etc/hosts
  echo "192.168.101.11 node-2" >> /etc/hosts
  echo "192.168.101.22 lb01" >> /etc/hosts
  echo "192.168.101.33 db01" >> /etc/hosts
  echo "192.168.102.10 ws-prod-01" >> /etc/hosts
  echo "192.168.102.11 ws-prod-02" >> /etc/hosts
  echo "192.168.102.21 lb-prod" >> /etc/hosts
  echo "192.168.102.31 db-prod" >> /etc/hosts
  #delete the own host line
  sed '/$hostIp/d' /etc/hosts

SCRIPT



masters=[

  {
    :hostname => "master-1",
    :ip => "192.168.100.10",
    :box => "debian/stretch64",
    :ram => 1024,
    :cpu => 1,
    :isProdEnv => false
  }
]

workers=[
  {
    :hostname => "node-1",
    :ip => "192.168.101.10",
    :box => "debian/stretch64",
    :ram => 512,
    :cpu => 1,
    :isProdEnv => false
  },
  {
    :hostname => "node-2",
    :ip => "192.168.101.11",
    :box => "debian/stretch64",
    :ram => 512,
    :cpu => 1,
    :isProdEnv => false
  },
  {
    :hostname => "lb01",
    :ip => "192.168.101.22",
    :box => "debian/stretch64",
    :ram => 512,
    :cpu => 1,
    :isProdEnv => false
  },
  {
    :hostname => "db01",
    :ip => "192.168.101.33",
    :box => "debian/stretch64",
    :ram => 512 ,
    :cpu => 1,
    :isProdEnv => false
  },
  {
    :hostname => "ws-prod-01",
    :ip => "192.168.102.10",
    :box => "debian/stretch64",
    :ram => 512,
    :cpu => 1,
    :isProdEnv => true
  },
  {
    :hostname => "ws-prod-02",
    :ip => "192.168.102.11",
    :box => "debian/stretch64",
    :ram => 512,
    :cpu => 1,
    :isProdEnv => true
  },
  {
    :hostname => "lb-prod",
    :ip => "192.168.102.21",
    :box => "debian/stretch64",
    :ram => 512,
    :cpu => 1,
    :isProdEnv => true
  },
  {
    :hostname => "db-prod",
    :ip => "192.168.102.31",
    :box => "debian/stretch64",
    :ram => 512 ,
    :cpu => 1,
    :isProdEnv => true
  }

]

Vagrant.configure(2) do |config|


  masters.each do |master|

    config.vm.define master[:hostname] do |master_config|
    
      master_config.vm.box = master[:box]
      master_config.vm.hostname = master[:hostname]
      master_config.vm.network "private_network", ip: master[:ip]
      master_config.vm.synced_folder "./working_directory", "/home/vagrant/working_directory", disabled:true
      master_config.vm.provision "file", source: "./ssh/key/master/id_rsa", destination: "/home/vagrant/.ssh/"   
      master_config.vm.provision "file", source: "./ssh/key/master/id_rsa.pub", destination: "/home/vagrant/.ssh/"     
      master_config.vm.provision "file", source: "./ssh/auth/authorized_keys_master", destination: "/home/vagrant/.ssh/"
      master_config.vm.provision :shell, :inline => "if [[ $(wc -l < /home/vagrant/.ssh/authorized_keys) -eq 1 ]]; then cat /home/vagrant/.ssh/authorized_keys_master >> /home/vagrant/.ssh/authorized_keys  ; fi ; rm -f /home/vagrant/.ssh/authorized_keys_master"

      master_config.vm.provision "shell", inline: $fix_ssh_permissions
      master_config.vm.provision "shell" do |s|
        s.inline = $rewrite_hosts
        s.args   = master[:ip]
      end
      master_config.vm.provision "shell", inline: $install_ansible
      master_config.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--memory", master[:ram]]
        vb.customize ["modifyvm", :id, "--cpus", master[:cpu]]
      end

    end
  end

  workers.each do |worker|

    config.vm.define worker[:hostname] do |worker_config|
    
      worker_config.vm.box = worker[:box]
      worker_config.vm.hostname = worker[:hostname]
      worker_config.vm.network "private_network", ip: worker[:ip]
      worker_config.ssh.forward_agent = true
      worker_config.vm.provision "shell", inline: "mkdir -p /home/vagrant/.ssh"
      if(worker[:isProdEnv])
        then 
          worker_config.vm.provision "file", source: "./ssh/key/node_prod/id_rsa", destination: "/home/vagrant/.ssh/"
          worker_config.vm.provision "file", source: "./ssh/key/node_prod/id_rsa.pub", destination: "/home/vagrant/.ssh/"
          worker_config.vm.provision "file", source: "./ssh/auth/authorized_keys_prod", destination: "/home/vagrant/.ssh/"
          worker_config.vm.provision :shell, :inline => "if [[ $(wc -l < /home/vagrant/.ssh/authorized_keys) -eq 1 ]]; then cat /home/vagrant/.ssh/authorized_keys_prod >> /home/vagrant/.ssh/authorized_keys  ; fi ; rm -f /home/vagrant/.ssh/authorized_keys_prod"
        else  
          worker_config.vm.provision "file", source: "./ssh/key/node_dev/id_rsa", destination: "/home/vagrant/.ssh/"
          worker_config.vm.provision "file", source: "./ssh/key/node_dev/id_rsa.pub", destination: "/home/vagrant/.ssh/"
          worker_config.vm.provision "file", source: "./ssh/auth/authorized_keys_dev", destination: "/home/vagrant/.ssh/"
          worker_config.vm.provision :shell, :inline => "if [[ $(wc -l < /home/vagrant/.ssh/authorized_keys) -eq 1 ]]; then cat /home/vagrant/.ssh/authorized_keys_dev >> /home/vagrant/.ssh/authorized_keys  ; fi ; rm -f /home/vagrant/.ssh/authorized_keys_dev"
      end
      
      worker_config.vm.provision "shell", inline: $fix_ssh_permissions 
      worker_config.vm.provision "shell" do |s|
        s.inline = $rewrite_hosts
        s.args   = worker[:ip]
      end

      worker_config.vm.provider "virtualbox" do |vb|
        vb.customize ["modifyvm", :id, "--memory", worker[:ram]]
        vb.customize ["modifyvm", :id, "--cpus", worker[:cpu]]
      end

    end
  end
  
end
