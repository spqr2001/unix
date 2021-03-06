# -*- mode: ruby -*-
# # vi: set ft=ruby :

require 'fileutils'

Vagrant.require_version ">= 1.6.0"

# Defaults for config options defined in CONFIG
$num_instances = 3
$instance_name_prefix = "centos"
$enable_serial_logging = false
$share_home = false
$vm_gui = false
$vm_memory = 3072
$vm_cpus = 2
$forwarded_ports = 22,80,443

# Attempt to apply the deprecated environment variable NUM_INSTANCES to
# $num_instances while allowing config.rb to override it
if ENV["NUM_INSTANCES"].to_i > 0 && ENV["NUM_INSTANCES"]
  $num_instances = ENV["NUM_INSTANCES"].to_i
end

# Use old vb_xxx config variables when set
def vm_gui
  $vb_gui.nil? ? $vm_gui : $vb_gui
end

def vm_memory
  $vb_memory.nil? ? $vm_memory : $vb_memory
end

def vm_cpus
  $vb_cpus.nil? ? $vm_cpus : $vb_cpus
end

Vagrant.configure("2") do |config|
  config.vm.provision "shell",
    inline: "echo assumeyes=1 >> /etc/yum.conf"  
end 



Vagrant.configure("2") do |config|
  config.vm.provision "shell",
    inline: "yum update && yum -y upgrade  && yum install -y yum-utils git ansible device-mapper-persistent-data lvm2 && 
yum-config-manager  --add-repo  https://download.docker.com/linux/centos/docker-ce.repo && 
yum-config-manager --disable docker-ce-nightly && 
yum -y install docker-ce docker-ce-cli containerd.io && 
systemctl enable docker && systemctl start docker"

end

Vagrant.configure("2") do |config|
  config.vm.provision "shell",
    inline: "git config --global user.email hofstaetter@casolution.de &&
git config --global user.name spqr2001 "
#https://github.com/spqr2001/unix.git && cd unix && 
#git remote set-url origin git@github.com:spqr2001/unix.git"
end

Vagrant.configure("2") do |config|
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "ansible/vagrant.yml"
  end
end




Vagrant.configure("2") do |config|
  # always use Vagrants insecure key
  config.ssh.insert_key = false


  config.vm.box = "centos/7"

  # enable hostmanager
  config.hostmanager.enabled = true

  # configure the host's /etc/hosts
  config.hostmanager.manage_host = true

  (1..$num_instances).each do |i|
    config.vm.define vm_name = "%s-%02d" % [$instance_name_prefix, i] do |config|
      config.vm.hostname = vm_name

      if $enable_serial_logging
        logdir = File.join(File.dirname(__FILE__), "log")
        FileUtils.mkdir_p(logdir)

        serialFile = File.join(logdir, "%s-serial.txt" % vm_name)
        FileUtils.touch(serialFile)

        config.vm.provider :virtualbox do |vb, override|
          vb.customize ["modifyvm", :id, "--uart1", "0x3F8", "4"]
          vb.customize ["modifyvm", :id, "--uartmode1", serialFile]
        end
      end

      # foward Docker registry port to host for node 01
      if i == 1
        config.vm.network :forwarded_port, guest: 5000, host: 5000
      end

      config.vm.provider :virtualbox do |vb|
        vb.gui = vm_gui
        vb.memory = vm_memory
        vb.cpus = vm_cpus
      end

      ip = "172.17.11.#{i+100}"
      config.vm.network :private_network, ip: ip

    end
  end
end

