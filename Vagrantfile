# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant::Config.run do |config|

  config.vm.box_url = "http://developer.nrel.gov/downloads/vagrant-boxes/CentOS-6.4-x86_64-v20130309.box"

  config.vm.define :ngircd do |ngircd_config|
    ngircd_config.vm.box = "ngircd"
    ngircd_config.vm.forward_port 6667, 6667
    ngircd_config.vm.provision :puppet do |ngircdpuppet|
     ngircdpuppet.manifests_path = "puppet/manifests"
     ngircdpuppet.module_path    = "puppet/modules"
     ngircdpuppet.options        = "--verbose --hiera_config /vagrant/hiera.yaml"
     ngircdpuppet.manifest_file  = "init.pp"
    end
  end
end

