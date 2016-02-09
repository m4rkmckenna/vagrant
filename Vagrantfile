# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.require_version ">= 1.8.0"
VAGRANTFILE_API_VERSION = 2
require 'yaml'

if ENV.has_key?('VAGRANT_CONFIG') then
  cfg =  YAML.load_file(ENV['VAGRANT_CONFIG'])
else
  cfg = YAML.load_file('config.yaml')
end

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  if cfg.has_key?('masters') then
    cfg['masters'].each do |master_config|
      if !master_config.has_key?('enabled') || master_config['enabled'] then
        config.vm.define master_config['hostname'] do |vm_def|
          configureVM(vm_def, master_config['hostname'], master_config, cfg['defaults'])
        end
      end
    end
  end
  if cfg.has_key?('minions') then
    cfg['minions'].each do |minion_config|
      if !minion_config.has_key?('enabled') || minion_config['enabled'] then
        config.vm.define minion_config['hostname'] do |vm_def|
          configureVM(vm_def, minion_config['hostname'], minion_config, cfg['defaults'])
        end
      end
    end
  end
end

def configureVM(vm_def, hostname, vm_config, defaults)
  vm_def.vm.hostname = hostname
  # SET Vagrant box
  if vm_config.has_key?('box_url') then
    vm_def.vm.box_url = vm_config['box_url']
  end
  if vm_config.has_key?('box') then
    vm_def.vm.box = vm_config['box']
  else
    vm_def.vm.box = defaults['box']
  end

  if vm_config.has_key?("ip") then
    vm_def.vm.network "private_network", ip: vm_config["ip"]
  end

  if vm_config.has_key?("port_forwards") then
    vm_config["port_forwards"].each do |port_forwards|
      vm_def.vm.network "forwarded_port", guest: port_forwards["guest"], host: port_forwards["host"], guest_ip: port_forwards["guest_ip"]
    end
  end

  if vm_config.has_key?("shell")
    vm_config["shell"].each do |cmd|
      vm_def.vm.provision "shell", privileged: false, inline: cmd, keep_color: true
    end
  end

  if vm_config.has_key?("sync_folders")
    vm_config["sync_folders"].each do |synced_folder|
      vm_def.vm.synced_folder synced_folder["source"], synced_folder["destination"]
    end
  end

  vm_def.vm.provider :virtualbox do |vb|
    vb.name = hostname
    vb.linked_clone = true
    #Configure RAM
    if vm_config.has_key?('ram') then
      vb.memory = vm_config['ram']
    else
      vb.memory = defaults['ram']
    end

    #Configure CPUs
    if vm_config.has_key?('cpus') then
      vb.cpus = vm_config['cpus']
    else
      vb.cpus = defaults['cpus']
    end
  end
  vm_def.vm.post_up_message = "Node [%s] has been started but may still be installing software" %  hostname
end
