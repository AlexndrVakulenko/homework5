# -*- mode: ruby -*-
# vim: set ft=ruby :
# home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"

MACHINES = {
  :lvm => {
        :box_name => "centos/7",
#        :box_version => "1804.02",
        :ip_addr => '192.168.56.102',
    :disks => {
        :sata1 => {
            :dfile => './lvm/sata1.vdi',
            :size => 10240,
            :port => 1
        },
        :sata2 => {
            :dfile => './lvm/sata2.vdi',
            :size => 2048, # Megabytes
            :port => 2
        },
        :sata3 => {
            :dfile => './lvm/sata3.vdi',
            :size => 1024, # Megabytes
            :port => 3
        },
        :sata4 => {
            :dfile => './lvm/sata4.vdi',
            :size => 1024,
            :port => 4
        }
    }
  },
}

Vagrant.configure("2") do |config|
#   config.vbguest.installer_options = { allow_kernel_upgrade: true }
#    config.vm.box_version = "1804.02"
    MACHINES.each do |boxname, boxconfig|
  
        config.vm.define boxname do |box|
  
            box.vm.box = boxconfig[:box_name]
            box.vm.host_name = boxname.to_s
  
            #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset
  
            box.vm.network "private_network", ip: boxconfig[:ip_addr]
  			box.vm.network "public_network", type: "dhcp", adapter: 2
			box.vm.synced_folder "vmshare/", "/vagrant"

            box.vm.provider :virtualbox do |vb|
                    vb.customize ["modifyvm", :id, "--memory", "2048"]
                    needsController = false
            boxconfig[:disks].each do |dname, dconf|
                unless File.exist?(dconf[:dfile])
                  vb.customize ['createhd', '--filename', dconf[:dfile], '--variant', 'Fixed', '--size', dconf[:size]]
                                  needsController =  true
                            end
  
            end
                    if needsController == true
                       vb.customize ["storagectl", :id, "--name", "SATA", "--add", "sata" ]
                       boxconfig[:disks].each do |dname, dconf|
                           vb.customize ['storageattach', :id,  '--storagectl', 'SATA', '--port', dconf[:port], '--device', 0, '--type', 'hdd', '--medium', dconf[:dfile]]
                       end
                    end
            end
  
        box.vm.provision "shell", inline: <<-SHELL
	      mkdir -p ~root/.ssh
          cp ~vagrant/.ssh/auth* ~root/.ssh
                repo_file=/etc/yum.repos.d/CentOS-Base.repo
                cp ${repo_file} ~/CentOS-Base.repo.backup
                sudo sed -i s/#baseurl/baseurl/ ${repo_file}
                sudo sed -i s/mirrorlist.centos.org/vault.centos.org/ ${repo_file}
                sudo sed -i s/mirror.centos.org/vault.centos.org/ ${repo_file}
                sudo yum clean all
                yum install -y mdadm smartmontools hdparm gdisk
          SHELL
  
        end
    end
  end