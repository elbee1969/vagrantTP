# -*- mode: ruby -*-
# vi: set ft=ruby :

############### INLINE SCRIPTS
@docker_install = <<SCRIPT
  apt install snapd
  service snapd start
  
  # install docker
  snap install docker
  snap start docker

  # config cle rsa pour jenkins
  cd ~/.ssh
  ssh-keygen -f jenkinsAgent
  cat jenkinsAgent.pub  >> authorized_keys
SCRIPT
###############

Vagrant.configure("2") do |config|

   config.vm.boot_timeout = 1200

   # serveur jenkins
   config.vm.define "jenkins" do |jenkins|
      jenkins.vm.box = "bento/ubuntu-16.04"
      jenkins.vm.network :private_network, ip: "192.168.5.1"
      jenkins.vm.hostname = "jenkins" 
	  
      jenkins.vm.provider :virtualbox do |v|
         v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
         v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
         v.customize ["modifyvm", :id, "--memory", 2048]
         v.customize ["modifyvm", :id, "--name", "jenkins"]
         v.customize ["modifyvm", :id, "--cpus", "2"]
      end
  
      # copie des shells
      jenkins.vm.synced_folder "./scripts/", "/home/vagrant/scripts", owner: "vagrant", group: "vagrant",:mount_options => ["dmode=777", "fmode=766"]

      # provisionning ssh
      jenkins.vm.provision "shell", inline: <<-SHELL
         sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
		 sed -i -e "s/\r//g" /home/vagrant/scripts/*
         service ssh restart
      SHELL

      # provisionning docker
      jenkins.vm.provision "shell", inline: @docker_install
   end
		  
   # serveur de production
   config.vm.define "srvprod" do |srvprod|
      srvprod.vm.box = "bento/ubuntu-16.04"
      srvprod.vm.network :private_network, ip: "192.168.5.2"
      srvprod.vm.hostname = "srvprod"

      srvprod.vm.provider :virtualbox do |v|
         v.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
         v.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
         v.customize ["modifyvm", :id, "--memory", 2048]
         v.customize ["modifyvm", :id, "--name", "srvprod"]
         v.customize ["modifyvm", :id, "--cpus", "2"]
      end
	  
      # provisionning ssh
      srvprod.vm.provision "shell", inline: <<-SHELL
         sed -i 's/ChallengeResponseAuthentication no/ChallengeResponseAuthentication yes/g' /etc/ssh/sshd_config
         service ssh restart
      SHELL

      # provisionning docker
      srvprod.vm.provision "shell", inline: @docker_install
   end

end
  
