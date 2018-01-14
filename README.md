# raspberrypi
configured raspberrypi cross compile toolchain based on host computer (Ubuntu 16.04) raspberrypi 1

more information : https://github.com/raspberrypi/tools
reference:
[1]https://github.com/Yadoms/yadoms/wiki/Cross-compile-for-raspberry-PI
[2]http://blog.shahada.abubakar.net/post/raspberry-pi-software-development-with-a-cross-compiler
[3]https://stackoverflow.com/questions/16040128/hook-up-raspberry-pi-via-ethernet-to-laptop-without-router
[4]https://medium.com/@au42/the-useful-raspberrypi-cross-compile-guide-ea56054de187
[5]https://stackoverflow.com/questions/21113434/error-cross-compiling-cpp-netlib-for-raspbian

#Prerequisites
on host pc(Ubuntu) : 
    sudo apt-get update
    sudo apt-get upgrade
    sudo apt-get -y install libboost-all-dev
    sudo apt-get -y install libtinyxml2-dev
    sudo apt-get -y install libi2c-dev
    sudo apt-get -y install libsqlite3-dev
    sudo apt-get -y install libcgicc5-dev
    sudo apt-get -y install libcurl4-openssl-dev
    sudo apt-get -y install libfcgi-dev
    sudo apt-get -y install libpcre++-dev
    sudo apt-get -y install wiringpi

#Setup Cross Compiler
cd ~/
git clone 

in terminal: create the following symlinks:
ln -s $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/lib/arm-linux-gnueabihf $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/lib/arm-linux-gnueabihf/4.9.3

ln -s $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/lib/arm-linux-gnueabihf $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/lib/arm-linux-gnueabihf/4.9.3

#Example(hier we have already compile)
#cross compile boost
download newest boost library(boost_1_66_0.tar.bz2)
tar --bzip2 -xf /path/to/boost_1_66_0.tar.bz2
./bootstrap.sh --prefix=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr
./b2 --with-atomic --with-chrono --with-date_time --with-filesystem --with-regex --with-serialization --with-thread --with-system --no-samples --no-tests toolset=gcc-arm link=static cxxflags=-fPIC
./b2 install

#cross compile OpenSSL 1.0.2n
Download OpenSSL and extract it
    Version 1.0.x is working.
    Version 1.1.x is not working well
wget https://www.openssl.org/source/openssl-1.0.2l.tar.gz
tar xzf openssl-1.0.2n.tar.gz
cd openssl-1.0.2n
./Configure --prefix=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr os/compiler:arm-linux-gnueabihf
make CC="arm-linux-gnueabihf-gcc" AR="arm-linux-gnueabihf-ar r" RANLIB="arm-linux-gnueabihf-ranlib"
sudo make install

#cross compile cpp-netlib
download cpp-netlib 0.12.0-final.tar.gz (cpp-netib 0.13.0 is not stable)
tar xzf cpp-netlib 0.12.0-final.tar.gz
cd cpp-netlib
mkdir build
cd build
cmake prefix=arm-linux-gnueabihf- path=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/bin -DCMAKE_TOOLCHAIN_FILE=$HOME/raspberrypi/pi.cmake -DOPENSSL_ROOT_DIR=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/local/ssl -DCMAKE_SKIP_BUILD_RPATH=FALSE -DCMAKE_BUILD_WITH_INSTALL_RPATH=FALSE -DCMAKE_INSTALL_PREFIX:PATH=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr -DCMAKE_INSTALL_RPATH_USE_LINK_PATH=TRUE ../
make
sudo make install

p.s. if we get error cpp-netlib/include/boost/network/uri/builder.hpp:9:10: fatal error: 'asio/ip/address.hpp' file not found
Manually copying $SRCDIR/deps/asio/asio/include/asio to $CMAKE_INSTALL_PREFIX/include/asio fixes the problem.

#cross compile PiNet
cd ~/PiNet
mkdir raspberrypi_build
cd raspberrypi_build
cmake prefix=arm-linux-gnueabihf- path=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/bin -DCMAKE_TOOLCHAIN_FILE=$HOME/raspberrypi/pi.cmake -DOPENSSL_ROOT_DIR=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/local/ssl ../src
make

# secure copy built PiNet to raspberrypi 
scp PiNet pi@10.42.0.212:/home/pi/PiNet/build
(default password:raspberry)
# test output of PiNet
ssh pi@10.42.0.212 ./PiNet/build/PiNet


