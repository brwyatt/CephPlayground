# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2" 

# Create local caching for packages
def local_cache(box_name)
  cache_dir = File.join(File.dirname(__FILE__), '.vagrant_cache', 'apt', box_name)
  partial_dir = File.join(cache_dir, 'partial')
  FileUtils.mkdir_p(partial_dir) unless File.exists? partial_dir
  cache_dir
end

# Create disk
def create_disk(box_name, disk_num)
  disk_path = File.join(File.dirname(__FILE__), '.ceph_storage', box_name, "#{disk_num}.vdi")
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "velocity42/xenial64"
  # Work around cosmetic issue (default uses -l which misbehaves when not in interactive shell)
  config.ssh.shell = "/bin/bash"

  #config.r10k.puppet_dir = "puppet"
  #config.r10k.puppetfile_path = "Puppetfile"
  #config.r10k.module_path = "puppet/modules"

  cache_dir = local_cache(config.vm.box)
  config.vm.synced_folder cache_dir, "/var/cache/apt/archives/"

  (0..2).each do |num|
    ip = num + 10
    config.vm.define "ceph#{num}" do |server|
      server.vm.hostname = "ceph#{num}"
      server.vm.provider "virtualbox" do |vb|
        vb.memory = 2048
        vb.cpus = 2

        vb.customize ["storagectl", :id, "--name", "SATA Controller", "--add", "sata"]

	(0..1).each do |disk_num|
	  disk = File.join(File.dirname(__FILE__), '.ceph_storage', "ceph#{num}", "#{disk_num}.vdi")
	  unless File.exist?(disk)
	    vb.customize ['createhd', '--filename', disk, '--size', 4 * 1024]
	  end
	  vb.customize ['storageattach', :id, '--storagectl', 'SATA Controller', '--port', disk_num+1, '--device', 0, '--type', 'hdd', '--medium', disk]
	end
      end
      server.vm.network "private_network", ip: "10.10.1.#{ip}", virtualbox__intnet: "internal"
      server.vm.provision "shell", path: "ceph_preflight.sh"
    end
  end

  config.vm.define "cephadmin" do |server|
    server.vm.hostname = "cephadmin"
    server.vm.network "private_network", ip: "10.10.1.5", virtualbox__intnet: "internal"
    server.vm.provision "shell", path: "ceph_preflight.sh", env: { "cephadmin": "true" }
  end

  config.hostmanager.enabled = true
  config.hostmanager.manage_host = false
  config.hostmanager.ignore_private_ip = false
  config.hostmanager.include_offline = true

end
