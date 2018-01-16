# raspberrypi cross compiler
configured raspberrypi cross compile toolchain based on host computer (Ubuntu 16.04) raspberrypi 1 <br />

# Prerequisites <br />
on raspberrypi :  <br />
    sudo apt-get update <br />
    sudo apt-get upgrade <br />
    sudo apt-get -y install libboost-all-dev <br />
    sudo apt-get -y install libtinyxml2-dev  <br />
    sudo apt-get -y install libi2c-dev <br />
    sudo apt-get -y install libsqlite3-dev <br />
    sudo apt-get -y install libcgicc5-dev <br />
    sudo apt-get -y install libcurl4-openssl-dev <br /> 
    sudo apt-get -y install libfcgi-dev <br />
    sudo apt-get -y install libpcre++-dev <br />
    sudo apt-get -y install wiringpi <br />

# Setup Cross Compiler
cd ~/ <br />
git clone https://github.com/sammysun0711/raspberrypi_cross_compiler.git <br />

in terminal: create the following symlinks: <br />
ln -s $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/lib/arm-linux-gnueabihf $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/lib/arm-linux-gnueabihf/4.9.3
<br />
ln -s $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/lib/arm-linux-gnueabihf $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/lib/arm-linux-gnueabihf/4.9.3
<br />
# Example(already compiled)
# cross compile boost
download newest boost library(boost_1_58_0.tar.bz2) <br />
tar --bzip2 -xf /path/to/boost_1_58_0.tar.bz2 <br />
./bootstrap.sh --prefix=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/local <br />
nano ~/user-config.jam <br />
        using gcc : arm : arm-linux-gnueabihf-g++ ;  <br />
./b2 --with-atomic --with-chrono --with-date_time --with-filesystem --with-regex --with-serialization --with-thread --with-system --no-samples --no-tests toolset=gcc-arm link=static cxxflags=-fPIC <br />
./b2 install <br />
rm ~/user-config.jam
# cross compile OpenSSL 1.0.2n
Download OpenSSL and extract it <br />
    Version 1.0.x is working. <br />
    Version 1.1.x is not working well <br />
wget https://www.openssl.org/source/openssl-1.0.2l.tar.gz <br />
tar xzf openssl-1.0.2n.tar.gz <br />
cd openssl-1.0.2n <br />
./Configure --prefix=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/local os/compiler:arm-linux-gnueabihf <br />
make CC="arm-linux-gnueabihf-gcc" AR="arm-linux-gnueabihf-ar r" RANLIB="arm-linux-gnueabihf-ranlib" <br />
sudo make install <br />

# cross compile cpp-netlib
download cpp-netlib 0.12.0-final.tar.gz (cpp-netib 0.13.0 is not stable) <br />
tar xzf cpp-netlib 0.12.0-final.tar.gz <br />
cd cpp-netlib <br />
mkdir build <br />
cd build <br />
cmake prefix=arm-linux-gnueabihf- path=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/bin -DCMAKE_TOOLCHAIN_FILE=$HOME/raspberrypi/pi.cmake -DOPENSSL_ROOT_DIR=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/local/ssl -DCMAKE_SKIP_BUILD_RPATH=FALSE -DCMAKE_BUILD_WITH_INSTALL_RPATH=FALSE -DCMAKE_INSTALL_PREFIX:PATH=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/local -DCMAKE_INSTALL_RPATH_USE_LINK_PATH=TRUE ../cpp-netlib <br />
make <br />
sudo make install <br />

p.s. if we get error cpp-netlib/include/boost/network/uri/builder.hpp:9:10: fatal error: 'asio/ip/address.hpp' file not found
Manually copying $SRCDIR/deps/asio/asio/include/asio to $CMAKE_INSTALL_PREFIX/include/asio fixes the problem. <br />

# cross compile PiNet
cd ~/PiNet <br />
mkdir raspberrypi_build <br />
cd raspberrypi_build <br />
cmake prefix=arm-linux-gnueabihf- path=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/bin -DCMAKE_TOOLCHAIN_FILE=$HOME/raspberrypi/pi.cmake -DOPENSSL_ROOT_DIR=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/local/ssl ../src  <br />
make <br />

# secure copy built PiNet to raspberrypi 
scp PiNet pi@10.42.0.212:/home/pi/PiNet/build <br />
(default password:raspberry)
# test output of PiNet
ssh pi@10.42.0.212 ./PiNet/build/PiNet <br />

# more detail information see reference:
[1]https://github.com/raspberrypi/tools <br />
[2]https://github.com/Yadoms/yadoms/wiki/Cross-compile-for-raspberry-PI <br />
[3]http://blog.shahada.abubakar.net/post/raspberry-pi-software-development-with-a-cross-compiler <br />
[4]https://stackoverflow.com/questions/16040128/hook-up-raspberry-pi-via-ethernet-to-laptop-without-router <br />
[5]https://medium.com/@au42/the-useful-raspberrypi-cross-compile-guide-ea56054de187 <br />
[6]https://stackoverflow.com/questions/21113434/error-cross-compiling-cpp-netlib-for-raspbian <br />
