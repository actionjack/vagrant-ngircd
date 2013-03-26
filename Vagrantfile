# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant::Config.run do |config|

  config.vm.box_url = "http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.4-x86_64-v20130309.box"

  config.vm.define :nircd do |nircd_config|
    nircd_config.vm.box = "nircd"
    nircd_config.vm.forward_port 6667, 6667
    nircd_config.vm.provision :puppet do |nircdpuppet|
     nircdpuppet.manifests_path = "puppet/manifests"
     nircdpuppet.module_path    = "puppet/modules"
     nircdpuppet.options        = "--verbose"
     nircdpuppet.manifest_file  = "init.pp"
    end
  end
end

