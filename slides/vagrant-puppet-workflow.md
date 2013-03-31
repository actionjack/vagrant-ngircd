!SLIDE 

# Vagrant workflow with Puppet

## Infrastructure as Code in a Sandbox

!SLIDE bullets transition=fade

# What is vagrant?
## A tool for creating and configuring work environments that are:
* Lightweight
* Portable
* Reproducible

!SLIDE bullets transition=fade

# Pre-requisite Virtualisation Layers:
* Virtual Box
* Vmware Fusion
* Amazon EC2

!SLIDE bullets transition=fade

# Install vagrant
* http://www.vagrantup.com
* http://downloads.vagrantup.com
* http://www.vagrantbox.es
* http://docs.vagrantup.com

!SLIDE transition=fade

# Bootstrap your Environment

```bash
$ mkdir -p ~/vagrant/vagrant-ngircd
$ cd ~/vagrant/vagrant-ngircd
$ vagrantbox=`pwd`
$ mkdir -p puppet/manifests
$ vagrant init

A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```
And that's it you can fire it up running

```bash
$ vagrant up
Bringing machine 'base' up with 'virtualbox' provider...
```

!SLIDE transition=fade

# Create a minimal puppetized Vagrantfile

```ruby
# -*- mode: ruby -*-
# vi: set ft=ruby :
  Vagrant::Config.run do |config|
    config.vm.box_url = "http://puppet-vagrant-boxes.puppetlabs.com/centos-63-x64.box"
    config.vm.define :ngircd do |config|
      config.vm.box = "ngircd"
      config.vm.forward_port 6667, 6667
      config.vm.provision :puppet do |puppet|
        puppet.manifests_path = "puppet/manifests"
        puppet.module_path    = "puppet/modules"
        puppet.options        = "--verbose"
        puppet.manifest_file  = "init.pp"
      end
    end
  end
```

!SLIDE transition=fade

# Let's break it down

## For my Vagrant configuration

<pre>
 Get my Virtual Machine Template from the following URL:
  http://puppet-vagrant-boxes.puppetlabs.com/centos-63-x64.box  
</pre>

## Let me define my virtual machine configuration:

<pre>
   I want to call it "ngircd",
   I would like port 6667 on my vm to be forwarded to port 6667 on my PC, 
   I use Puppet as my config management tool with the following options:
    my puppet manifest is in the $vagrantbox/puppet/manifests directory,
    my puppet modules are in the $vagrantbox/puppet/modules directory,
    I want my puppet execution run to produce verbose output,
    my puppet manifest file is called init.pp.
</pre>

!SLIDE transition=fade

# Libraries & Linting

```bash
$ gem install librarian-puppet puppet-lint
Successfully installed librarian-puppet-0.9.8
Successfully installed puppet-lint-0.3.2
2 gems installed
```
## Librarian-puppet
### A bundler for puppet modules. 
### Resolves, fetches, installs & isolate a project's dependencies.
### http://librarian-puppet.com

## Puppet-lint
### A linting tool
### Checks that your Puppet manifest conforms to the style guide
### http://puppet-lint.com

!SLIDE transition=fade

# Fetch your module bundle

## Initialize librarian-puppet

```bash
$ cd $vagrantbox/puppet
$ librarian-puppet init
$ vi PuppetFile
```
<pre>
forge "http://forge.puppetlabs.com"
mod 'puppetlabs/stdlib'
mod 'ntp',
   :git => 'git://github.com/puppetlabs/puppetlabs-ntp.git'
</pre>

## Install modules

```bash
$ librarian-puppet install
```

!SLIDE transition=fade 

# Hack your puppet module
## Clone or create your module
```bash
$ cd $vagrantbox/puppet/modules 
puppet module generate actionjack-ngircd
Notice: Generating module at /tmp/actionjack-ngircd
actionjack-ngircd
actionjack-ngircd/Modulefile
actionjack-ngircd/README
actionjack-ngircd/manifests
actionjack-ngircd/manifests/init.pp
actionjack-ngircd/spec
actionjack-ngircd/spec/spec_helper.rb
actionjack-ngircd/tests
actionjack-ngircd/tests/init.pp
```
## Validate and lint

```bash
$ puppet parser validate init.pp
$ puppet-lint --with-filename .
```

!SLIDE transition=fade

# Fix up your manifest file

```bash
$ vi $vagrantbox/puppet/manifests/init.pp
```
```ruby
node default {
  Yumrepo['epel'] ->
  Class['ntp'] ->
  Class['ngircd']

  yumrepo { 'epel':
    descr          => ' Extra Packages for Enterprise Linux 6 - $basearch',
    mirrorlist     => 'https://mirrors.fedoraproject.org/metalink?repo=epel-6&arch=$basearch',
    failovermethod => 'priority',
    enabled        => $enabled,
    gpgcheck       => $enabled,
    gpgkey         => 'http://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-6',
  }
  include ntp
  class{ 'ngircd':
    listenaddress => '0.0.0.0'
  }
}
```


!SLIDE transition=fade
# Firing it all up

```bash
$ cd $vagrantbox
$ vagrant up
Bringing machine 'ngircd' up with 'virtualbox' provider...
[ngircd] Importing base box 'ngircd'...
...
[ngircd] Forwarding ports...
[ngircd] -- 22 => 2222 (adapter 1)
[ngircd] -- 6667 => 6667 (adapter 1)
[ngircd] Booting VM...
[ngircd] Waiting for VM to boot. This can take a few minutes.
[ngircd] VM booted and ready for use!
[ngircd] Configuring and enabling network interfaces...
[ngircd] Mounting shared folders...
[ngircd] -- /vagrant
[ngircd] -- /tmp/vagrant-puppet/manifests
[ngircd] -- /tmp/vagrant-puppet/modules-0
[ngircd] Running provisioner: VagrantPlugins::Puppet::Provisioner::Puppet...
Running Puppet with init.pp...
...
Notice: Finished catalog run in 64.75 seconds
```
	
!SLIDE transition=fade

# Did it all work? (Y/N)
```bash
$ gedit $vagrantbox/puppet/module/ngircd/manifests/init.pp
$ vagrant provision
```
## Hack around a little more..
```bash
$ pushd $vagrantbox/puppet/module/ngircd/manifests
$ puppet parser validate *.pp
$ puppet-lint --with-filename *.pp
$ popd
$ vagrant provision
```

## Working? 
```bash
$ pushd $vagrantbox/puppet/module/ngircd
$ git commit -m "Made X change" -a
$ git push
```
!SLIDE transition=fade

# Does it really work from scratch?

## Does everything work cleanly on a new build

```bash
$ vagrant destroy -f && vagrant up
[ngircd] Forcing shutdown of VM...
[ngircd] Destroying VM and associated drives...
Bringing machine 'ngircd' up with 'virtualbox' provider...
[ngircd] Importing base box 'ngircd'...
```
## NB: Release Confidence
### Avoid the anti-pattern of having to run puppet X times to reach a known state

!SLIDE transition=fade

# Q & A?

## The Vagrant environment and slides for this presentation can be found here: 
### https://github.com/actionjack/vagrant-ngircd.git

## More Automated Test Driven Infrastructure
### Unit Testing using [rspec-puppet](http://rspec-puppet.com)
### TDI your Infrastructure using [Cumberbatch](https://github.com/actionjack/cumberbatch)
### Continuous Integration using [Travis CI](https://travis-ci.org) for puppet modules
