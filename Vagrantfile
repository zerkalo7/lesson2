# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']

MACHINES = {
  :otuslinux => {
        :box_name => "centos/7",
        :ip_addr => '192.168.11.101',
	:disks => {
		:sata1 => {
			:dfile => home + '/VirtualBox VMs/Raid/lvm_sata1.vdi',
			:size => 40960,
			:port => 1
#		},
#		:sata2 => {
#                       :dfile => home + '/VirtualBox VMs/Raid/lvm_sata2.vdi',
#                        :size => 250, # Megabytes
#			:port => 2
#		},
#                :sata3 => {
#                        :dfile => home + '/VirtualBox VMs/Raid/lvm_sata3.vdi',
#                        :size => 250,
#                        :port => 3
#                },
#                :sata4 => {
#                        :dfile => home + '/VirtualBox VMs/Raid/lvm_sata4.vdi',
#                        :size => 250, # Megabytes
#                        :port => 4
#                },
#		:sata5 => {
#			:dfile => home + '/VirtualBox VMs/Raid/lvm_sata5.vdi',
#			:size => 250, 
#			:port => 5
		}

	}

		
  },
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

      config.vm.define boxname do |box|

          box.vm.box = boxconfig[:box_name]
          box.vm.host_name = boxname.to_s

          #box.vm.network "forwarded_port", guest: 3260, host: 3260+offset

          box.vm.network "private_network", ip: boxconfig[:ip_addr]

          box.vm.provider :virtualbox do |vb|
            	  vb.customize ["modifyvm", :id, "--memory", "2048"]
		  vb.customize ["modifyvm", :id, "--cpus", "4"]
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
		yum install -y mdadm smartmontools hdparm gdisk nano wget tree xfsdump
#		mdadm -Cv /dev/md0 -l 5 -n 4 /dev/sd{c..f}
#		sudo mkdir /etc/mdadm
#		echo "DEVICE partitions" | sudo tee -a /etc/mdadm/mdadm.conf
#		sudo mdadm --detail -sv | awk '/ARRAY/{print}' | sudo tee -a /etc/mdadm/mdadm.conf
#		sudo parted -s /dev/md0 mklabel gpt
#		sudo parted /dev/md0 mkpart primary ext4 0% 20%
#		sudo parted /dev/md0 mkpart primary ext4 20% 40%
#		sudo parted /dev/md0 mkpart primary ext4 40% 60%
#		sudo parted /dev/md0 mkpart primary ext4 60% 80%
#		sudo parted /dev/md0 mkpart primary ext4 80% 100%
#		for i in $(seq 1 5); do sudo mkfs.ext4 /dev/md0p$i; done
#		sudo mkdir -p /raid/part{1..5}
#		for i in $(seq 1 5); do sudo mount /dev/md0p$i /raid/part$i; done 
  	  SHELL

      end
  end
end
