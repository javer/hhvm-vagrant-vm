# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

$build_script = <<SCRIPT
apt-get update

echo Installing HHVM dependencies...
apt-get install -y git-core cmake g++ libboost1.48-dev libmysqlclient-dev \
  libxml2-dev libmcrypt-dev libicu-dev openssl build-essential binutils-dev \
  libcap-dev libgd2-xpm-dev zlib1g-dev libtbb-dev libonig-dev libpcre3-dev \
  autoconf libtool libcurl4-openssl-dev libboost-regex1.48-dev libboost-system1.48-dev \
  libboost-program-options1.48-dev libboost-filesystem1.48-dev wget memcached \
  libreadline-dev libncurses-dev libmemcached-dev libbz2-dev \
  libc-client2007e-dev php5-mcrypt php5-imagick libgoogle-perftools-dev \
  libcloog-ppl0 libelf-dev libdwarf-dev subversion

echo Installing nginx, php and other useful tools...
apt-get install -y nginx-full \
  php5-cli php5-curl php5-fpm php5-gd php5-intl php5-json \
  php5-memcached php5-mysql php5-tidy php5-xsl php-apc \
  unzip apache2-utils
cp -R /vagrant/etc/* /etc/
chmod +x /etc/init.d/hhvm

echo Initializing site
mkdir -p /var/www
rm -rf /var/www/site
cp -R /vagrant /var/www/site
mkdir -p /var/www/site/web
mkdir -p /var/www/site/app/logs
chown -R vagrant:vagrant /var/www

cores=5
mkdir -p ~/dev
cd ~/dev
export CMAKE_PREFIX_PATH=`pwd`
export HPHP_HOME=`pwd`/hhvm

echo Getting HHVM source-code...
git clone git://github.com/facebook/hhvm.git

echo Building libevent...
git clone git://github.com/libevent/libevent.git
cd libevent
git checkout release-1.4.14b-stable
cat ../hhvm/hphp/third_party/libevent-1.4.14.fb-changes.diff | patch -p1
./autogen.sh
./configure --prefix=$CMAKE_PREFIX_PATH
make -j$cores
make install
cd ..

echo Building libCurl...
git clone git://github.com/bagder/curl.git
cd curl
./buildconf
./configure --prefix=$CMAKE_PREFIX_PATH
make -j$cores
make install
cd ..

echo Building Google glog...
svn checkout http://google-glog.googlecode.com/svn/trunk/ google-glog
cd google-glog
./configure --prefix=$CMAKE_PREFIX_PATH
make -j$cores
make install
cd ..

echo Building JEMalloc 3.0...
wget http://www.canonware.com/download/jemalloc/jemalloc-3.0.0.tar.bz2
tar xjvf jemalloc-3.0.0.tar.bz2
cd jemalloc-3.0.0
./configure --prefix=$CMAKE_PREFIX_PATH
make -j$cores
make install
cd ..

echo Building HHVM...
cd hhvm
cmake .
make -j$cores

echo Starting services...
/etc/init.d/nginx restart
/etc/init.d/php5-fpm restart
/etc/init.d/hhvm start
SCRIPT

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "precise64"
  config.vm.box_url = "http://files.vagrantup.com/precise64.box"
  config.vm.network :forwarded_port, guest: 80, host: 9080
  config.vm.network :forwarded_port, guest: 81, host: 9081

  config.vm.provider :virtualbox do |vb|
    vb.name = "HipHop VM"
    vb.customize ["modifyvm", :id, "--memory", "2048"]
    vb.customize ["modifyvm", :id, "--cpus", "4"]
    vb.customize ["modifyvm", :id, "--hwvirtex", "on"]
    vb.customize ["modifyvm", :id, "--nestedpaging", "on"]
  end

  config.vm.provision :shell, inline: $build_script
end
