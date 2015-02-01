# -*- mode: ruby -*-
# vi: set ft=ruby :

# Vagrantfile API/syntax version. Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

$build_script = <<SCRIPT
apt-get update

echo Installing HHVM dependencies...
apt-get install -y git-core cmake g++ libmysqlclient-dev \
  libxml2-dev libmcrypt-dev libicu-dev openssl build-essential binutils-dev \
  libcap-dev libgd2-xpm-dev zlib1g-dev libtbb-dev libonig-dev libpcre3-dev \
  autoconf automake libtool libcurl4-openssl-dev \
  wget memcached libreadline-dev libncurses-dev libmemcached-dev libbz2-dev \
  libc-client2007e-dev php5-mcrypt php5-imagick libgoogle-perftools-dev \
  libcloog-ppl0 libelf-dev libdwarf-dev subversion python-software-properties \
  libmagickwand-dev libxslt1-dev ocaml-native-compilers libevent-dev gawk

echo Upgrading gcc to 4.8
add-apt-repository ppa:ubuntu-toolchain-r/test
apt-get update
apt-get install -y gcc-4.8 g++-4.8
update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.8 60 \
                    --slave /usr/bin/g++ g++ /usr/bin/g++-4.8
update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-4.6 40 \
                    --slave /usr/bin/g++ g++ /usr/bin/g++-4.6
update-alternatives --set gcc /usr/bin/gcc-4.8

echo Installing Boost 1.49
add-apt-repository ppa:mapnik/boost
apt-get update
apt-get install -y libboost1.49-dev libboost-regex1.49-dev \
  libboost-system1.49-dev libboost-program-options1.49-dev \
  libboost-filesystem1.49-dev libboost-thread1.49-dev

echo Installing nginx, php and other useful tools...
apt-get install -y nginx-full \
  php5-cli php5-curl php5-fpm php5-gd php5-intl php5-json \
  php5-memcached php5-mysql php5-tidy php5-xsl php-apc \
  unzip apache2-utils
cp -R /vagrant/etc/* /etc/
chmod +x /etc/init.d/hhvm
mv /etc/php5/fpm/pool.d/www.conf /etc/php5/fpm/pool.d/www.conf.dist

echo Initializing site
mkdir -p /var/www
rm -rf /var/www/site
cp -R /vagrant /var/www/site
mkdir -p /var/www/site/web
mkdir -p /var/www/site/app/logs
chown -R vagrant:vagrant /var/www

cores=3
mkdir -p ~/dev
cd ~/dev
export CMAKE_PREFIX_PATH=`pwd`

echo Getting HHVM source-code...
git clone git://github.com/facebook/hhvm.git --depth=1

echo Building libCurl...
git clone git://github.com/bagder/curl.git --depth=1
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

echo Building JEMalloc 3.x...
wget http://www.canonware.com/download/jemalloc/jemalloc-3.5.1.tar.bz2
tar xjvf jemalloc-3.5.1.tar.bz2
cd jemalloc-3.5.1
./configure --prefix=$CMAKE_PREFIX_PATH
make -j$cores
make install
cd ..

echo Building HHVM...
cd hhvm
git submodule update --init --recursive
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
