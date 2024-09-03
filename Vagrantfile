# -*- mode: ruby -*-
# vi: set ft=ruby :
VAGRANTFILE_API_VERSION = "2"
Vagrant.require_version ">= 2.4.1"

require_relative 'secrets.rb'
include Secrets

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  vms = Secrets.vms

  cpus_default = 1
  ram_default = 1024
  rosetta_default = "off"
  isolate_default = "on"
  bridge_default = "en0"

  config.vm.box = Secrets.box
  config.ssh.username = Secrets.username
  config.ssh.insert_key = false
  config.ssh.forward_agent = true
  config.ssh.keys_only = false
  config.ssh.private_key_path = Secrets.private_key_path

  vms.each do |vm|
  config.vm.define vm[:name] do |server|
    server.vm.hostname = vm[:name]
    # server.ssh.host = "#{Secrets.base_ip_address}.#{IP[i]}"
    # server.ssh.host = vm[:name]
    server.vm.synced_folder ".", "/vagrant", disabled: true
    server.vm.provider "parallels" do |prl|
      prl.linked_clone = true
      prl.name = vm[:name]
      prl.check_guest_tools = false
      prl.update_guest_tools = false
      prl.cpus = vm[:cpus] || cpus_default
      prl.memory = vm[:ram] || ram_default
      prl.customize "post-import", ["set", :id, "--smart-mouse-optimize", "off"]
      prl.customize "post-import", ["set", :id, "--keyboard-optimize", "off"]
      prl.customize "post-import", ["set", :id, "--sync-host-printers", "off"]
      prl.customize "post-import", ["set", :id, "--autostart", "start-host"]
      prl.customize "post-import", ["set", :id, "--autostop", "suspend"]
      prl.customize "post-import", ["set", :id, "--autostart-delay", "30"]
      prl.customize "post-import", ["set", :id, "--time-sync", "off"]
      prl.customize "post-import", ["set", :id, "--rosetta-linux", "#{vm[:rosetta] || rosetta_default}"]
      prl.customize "post-import", ["set", :id, "--isolate-vm", "#{vm[:isolate] || isolate_default}"]
      prl.customize "post-import", ["set",
                                  :id,
                                  "--device-add" , "hdd",
                                  "--image", "#{Secrets.additional_disk_path}/#{vm[:name]}.hdd"]
      prl.customize "pre-boot", ["set", :id, "--device-del", "sound0"]
    end

    server.trigger.after :up do |trigger|
      trigger.ignore = :provision
      trigger.info = "Shutdown VM"
      trigger.run = {inline: "prlctl stop #{vm[:name]}"}
    end

    server.trigger.after :up do |trigger|
      trigger.ignore = :provision
      trigger.info = "Modify net0"
      trigger.run = {inline: "prlctl set #{vm[:name]} --device-set net0
                                                      --type bridged
                                                      --mac #{Secrets.base_mac_address}:#{vm[:ip].chars.last(2).join}"}
    end

    server.trigger.after :up do |trigger|
      trigger.ignore = :provision
      trigger.info = "Start VM"
      trigger.run = {inline: "prlctl start #{vm[:name]}"}
    end
  end
  end
end
