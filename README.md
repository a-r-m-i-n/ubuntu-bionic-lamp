# Ubuntu Bionic Beaver LAMP Vagrant Box

The [ArminVieweg/ubuntu-bionic64-lamp](https://app.vagrantup.com/ArminVieweg/boxes/ubuntu-bionic64-lamp) box 
is a lightweight (~800MB) Vagrant box for VirtualBox based on latest Ubuntu 18.04 LTS "Bionic Beaver" 64-bit. 
This box is released under GPL-2.0+. 

**The following components are installed:**

* Apache 2 (with SSL available)
* PHP 5.6, 7.0 - 7.3 (7.3 enabled by default)
* XDebug for all versions
* MariaDB
* ImageMagick
* Composer
* NodeJS & npm
* Ruby
* Mailcatcher


## Installation

Just perform a `vagrant init ArminVieweg/ubuntu-bionic64-lamp` in your project to use this box.
This will create a very basic Vagrantfile for you.

You may also use the Vagrantfile in this repository. It ships a pretty solid configuration which should 
work out of the box. Put the file to your project's root and perform a `vagrant up`.


### Vagrant Plugins

The Vagrantfile from this repo requires the following plugins to be installed:

* vagrant-bindfs
* vagrant-hostmanager
* vagrant-winnfsd (for Windows only)

To install a Vagrant plugin simply enter

```
$ vagrant plugin install <name>
```

### Variables in Vagrantfile

There are some points in Vagrantfile you may adjust for your projects. To be able to run multiple instances
on one host, you need to specify a unique static ip address. 

Default values:

```
staticIpAddress = "192.168.12.2"
config.vm.hostname = "bionic.local"
```

### Provisioning scripts

Vagrantfile comes with a provisioning scripts, which updates the SSL certificate for apache, 
based on configured hostname.


## Basics

* The default document root is `/var/www/html`
* In MySQL/MariaDB use `root`/`root` to login
* For SSH use port 22 and credentials `vagrant`/`vagrant`. You can also use the `private_key` in .vagrant folder.
* To open Mailcatcher use port 1080: http://bionic.local:1080
* Supports `vagrant share` (with [ngrok](https://ngrok.com/download) installed)


## Configuration

In home directory of Vagrant user are some php.ini files located, which are sym-linked in conf.d folders of php versions.
The following files are existing:

* **php-5.6.ini** Just used with PHP 5.6
* **php-7.0.ini** Just used with PHP 7.0
* **php-7.1.ini** Just used with PHP 7.1
* **php-7.2.ini** Just used with PHP 7.2
* **php-7.3.ini** Just used with PHP 7.3
* **php-all.ini** Used with all versions (not CLI)
* **php-cli.ini** Used for CLI
* **php-xdebug.ini** Settings for XDebug (remote debugging enabled by trigger, profiling prepared but disabled by default)

The paths `/etc/apache2`, `/etc/php` and `/etc/mysql` have got write permissions for Vagrant user. 
So you can edit configuration with e.g. WinSCP, without need to perform `sudo` on CLI. 

### Switch PHP version

To switch the current PHP version just execute:

```
$ sudo ~/enable-php/7.2.sh
```

Available versions are:

* **5.6.sh**
* **7.0.sh**
* **7.1.sh**
* **7.2.sh**
* **7.3.sh** (enabled by default)

Apache server will get automatically restarted and also the CLI PHP version will get changed.


## Disk space

The virtual hard disk shipped with [ArminVieweg/ubuntu-bionic64-lamp](https://app.vagrantup.com/ArminVieweg/boxes/ubuntu-bionic64-lamp) 
stores up to 500 GB by default.

New instances of this box require ~2GB of disk space.


## Pro Tips

### Mount composer cache

As a PHP developer I'm working very much with the dependency manager Composer. A disadvantage of working with virtual
machines, are the separated disks each VM has. So composer will always download dependencies when executed first time
in VM. 

To avoid that, you can create a Vagrantfile in **.vagrant.d** folder in your home directory, with this content:

```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
    # Mounting Composer cache to all VMs
    config.vm.synced_folder "/path/to/composer/home/cache", "/var/nfs-composer-cache", type: "nfs"
    config.bindfs.bind_folder "/var/nfs-composer-cache", "/home/vagrant/.cache/composer", perms: "u=rwX:g=rwX:o=rD"
end
```
This will mount your local composer cache, on host, to VM and every cached package will be installed instantly.
