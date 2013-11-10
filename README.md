HipHop-Vagrant-VM
=================

Description
-----------

Setup a test environment for HipHop VM.

The final VM will contain HHVM, Nginx, PHP, MySQL, ab.

Requirements
------------

* [VirtualBox](https://www.virtualbox.org)
* [Vagrant 1.2.x](http://vagrantup.com)

Installation
------------

Clone the repository:

```bash
$ git clone https://github.com/javer/hhvm-vagrant-vm
$ cd hhvm-vagrant-vm
```

And copy your project source into this folder:

```bash
$ cp -R /var/www/site/* ./
```

Finally, you should run:

```bash
$ vagrant up
```

By default the VM uses 2GB of memory and 4 CPU core. If you want to use more memory or cores you can edit this settings directly in the Vagrantfile.

If something goes wrong (i.e. insufficient memory when compiling HHVM), you can log into the virtual machine and run building HHVM manually:
```bash
$ vagrant ssh
$ sudo -s
$ cd /root/dev/hhvm
$ export CMAKE_PREFIX_PATH=`pwd`/..
$ export HPHP_HOME=`pwd`
$ cmake .
$ make -j5
```

Once everything is done you can log into the virtual machine, setup your project in /var/www/site folder:

```bash
$ vagrant ssh
$ cd /var/www/site
$ ...
```

Start HHVM:
```bash
$ sudo /etc/init.d/hhvm start
```

Now you can browse your project on the host machine by entering http://127.0.0.1:9080 for serving site using PHP-FPM and http://127.0.0.1:9081 for serving site using HHVM.

Virtual Machine Management
--------------------------

When done just log out with `^D` and suspend the virtual machine

```bash
$ vagrant suspend
```

then, resume to testing again

```bash
$ vagrant resume
```

Run

```bash
$ vagrant halt
```

to shutdown the virtual machine, and

```bash
$ vagrant up
```

to boot it again.

You can find out the state of a virtual machine anytime by invoking

```bash
$ vagrant status
```

Finally, to completely wipe the virtual machine from the disk **destroying all its contents**:

```bash
$ vagrant destroy
```

Information
-----------

* Virtual Machine user/password: vagrant/vagrant
* Serving site using PHP: http://127.0.0.1:9080/
* Serving site using HHVM: http://127.0.0.1:9081/
* SSH into your vm using your favorite ssh client: ssh://127.0.0.1:2222 (username+password: "vagrant")
* Start nginx: `sudo /etc/init.d/nginx start`
* Start php-fpm: `sudo /etc/init.d/php5-fpm start`
* Start HHVM: `sudo /etc/init.d/hhvm start`

Updating
--------

For building latest version of HHVM you should run the following command on the host machine:

```bash
$ vagrant destroy && vagrant up
```
