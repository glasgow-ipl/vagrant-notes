# -*- mode: ruby -*-
# vi: set ft=ruby :


$PROVISION = <<SCRIPT
	echo "VM Setup Completed"
SCRIPT


Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2004"
  
  # Uncomment the following to enable X11 forwarding
  #config.ssh.forward_agent = true
  #config.ssh.forward_x11 = true


  config.vm.provider :libvirt do |libvirt|
    libvirt.memory=16384
    libvirt.cpus=8
    libvirt.driver="qemu"
  end

  config.vm.provision "shell", privileged: true, inline: $PROVISION
  
  config.vm.synced_folder ".", "/vagrant", type: "nfs", nfs_version: 4, nfs_udp: false

end
