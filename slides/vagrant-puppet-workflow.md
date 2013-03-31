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
$ mkdir -p ~/vagrant/vagrant-nircd
$ cd ~/vagrant/vagrant-nircd
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
    config.vm.define :nircd do |config|
      config.vm.box = "nircd"
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
   I want to call it "nircd",
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
### Checks that your Puppet manifest conform to the style guide
### http://puppet-lint.com

!SLIDE transition=fade

# Fetch your module bundle
Initialize librarian-puppet

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

and install modules

```bash
$ librarian-puppet install
```

!SLIDE transition=fade 

# Hack your puppet module

Clone or create your module

```bash
$ cd $vagrantbox/puppet/modules 
$ git clone git@github.com:uncommonsense/puppet-ngircd.git ngircd
$ cd ngircd
$ git checkout industrialisation
$ git pull origin industrialisation
$ cd manifest
$ vi init.pp
```
Validate and lint

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
Bringing machine 'nircd' up with 'virtualbox' provider...
[nircd] Importing base box 'nircd'...
...
[nircd] Forwarding ports...
[nircd] -- 22 => 2222 (adapter 1)
[nircd] -- 6667 => 6667 (adapter 1)
[nircd] Booting VM...
[nircd] Waiting for VM to boot. This can take a few minutes.
[nircd] VM booted and ready for use!
[nircd] Configuring and enabling network interfaces...
[nircd] Mounting shared folders...
[nircd] -- /vagrant
[nircd] -- /tmp/vagrant-puppet/manifests
[nircd] -- /tmp/vagrant-puppet/modules-0
[nircd] Running provisioner: VagrantPlugins::Puppet::Provisioner::Puppet...
Running Puppet with init.pp...
...
Notice: Finished catalog run in 64.75 seconds
```
	
!SLIDE transition=fade

# Did it all work? (Y/N)
```bash
$ vagrant ssh
$ sudo su -
$ cd /tmp/vagrant-puppet/manifests 
$ puppet apply --verbose --modulepath '/tmp/vagrant-puppet/modules-0' init.pp
```
Hack around a little more..

```bash
$ cd /tmp/vagrant-puppet/manifests
$ puppet apply --verbose --modulepath '/tmp/vagrant-puppet/modules-0' init.pp
```

Hack about a little more..

```bash
$ cd /tmp/vagrant-puppet/manifests
$ puppet apply --verbose --modulepath '/tmp/vagrant-puppet/modules-0' init.pp
```
Working! (Don't forget to validate and lint!)

!SLIDE transition=fade

# Does it really work from scratch?

Does everything work cleanly on a new build

```bash
$ vagrant destroy -f && vagrant up
[nircd] Forcing shutdown of VM...
[nircd] Destroying VM and associated drives...
Bringing machine 'nircd' up with 'virtualbox' provider...
[nircd] Importing base box 'nircd'...
```
NB: Release Confidence - Avoid the anti-pattern of having to run puppet X times to reach a known state

!SLIDE transition=fade

# Questions and a little live demo perhaps?

## Vagrant and Puppet Environment is available here:

https://github.com/actionjack/vagrant-nircd.git

## Future plans 

TDD your Infrastructure using [Cumberbatch](https://github.com/actionjack/cumberbatch)
