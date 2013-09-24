---
layout: post
title: "Dev-ops from 0 to Heroku"
date: 2013-09-23 16:26
published: false

---

Vagrant Specific (this is for your local box)

```
# add a new box, precise64 (12.04) is the latest Ubuntu LTS.
vagrant box add precise64 http://files.vagrantup.com/precise64.box

vagrant box list

```

Setting up the environment (on your vagrant/production box)
Goals -

- Bootstrap vagrant instance with Berkshelf
-  Install git, ruby, bundler, nginx
-  Get the static site out
Machine Setup

We will make use of Berkshelf to manage cookbooks. More info here:

```
# Berkshelf is pretty much bundler but for cookbooks
gem install berkshelf

# This plugin allows berkshelf to provision your vagrant box using the vagrant commands
vagrant plugin install vagrant-berkshelf

# The vagrant-omnibus plugin hooks into Vagrant and allows you to specify the version of the Chef Omnibus package you want installed using the omnibus.chef_version key
vagrant plugin install vagrant-omnibus
```

After the plugins are installed, create your cookbook using:

```
# Create the cookbook skeleton
berks cookbook nilenso-cookbook

# Bundle your Gemfile
bundle

# Lock down the ruby version
rbenv local 2.0.0-p247
```

The Vagrant file that got generated needs to be edited. Remove the legacy `config.ssh.timeout` and `config.ssh.maxtries` and add `config.vm.boot_timeout` setting as it is deprecated in Vagrant 1.3.0. Berkshelf (ver 2.0.10) has not yet been updated to reflect this in the scaffold that it generates. Since precise64 is already installed, you can get rid of the box_url setting and keep the `box` setting as `precise64`. At the moment, you do not need `chef.json` too.

```
Vagrant.configure("2") do |config|
  config.vm.hostname = "nilenso-cookbook-berkshelf"
  config.vm.box = "precise64"
  config.vm.network :private_network, ip: "33.33.33.10"

  config.vm.timeout = 120

  config.berkshelf.enabled = true

  config.vm.provision :chef_solo do |chef|
    chef.run_list = [
        "recipe[nilenso-cookbook::default]"
    ]
  end
end

```