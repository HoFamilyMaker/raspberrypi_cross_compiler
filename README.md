# raspberrypi cross compiler
configured raspberrypi cross compile toolchain based on host computer (Ubuntu 16.04) raspberrypi 1 

## Prerequisites 
### on raspberrypi : <br />
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
### install raspidjson on raspberrypi (cross compile not very easy...)  
    	git clone https://github.com/Tencent/rapidjson.git 
    	cd rapidjson 
    	git submodule update --init 
    	mkdir build 
    	cd build 
    	cmake ../ 
    	make 
    	sudo make install 	

## Setup Cross Compiler
	cd ~/ 
	git clone https://github.com/sammysun0711/raspberrypi.git 
	cd raspberrypi 
	git clone https://github.com/raspberrypi/tools

### get /usr /lib /opt file via rsync from raspberrypi 
	sudo rsync -rl --delete-after --safe-links pi@10.42.0.212:/{opt,lib,usr} $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot

### create the following symlinks:
	sudo ln -s $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/lib/arm-	linux-gnueabihf $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/lib/arm-linux-gnueabihf/4.9.3
	
	sudo ln -s $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/lib/arm-linux-gnueabihf $HOME/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/lib/arm-linux-	gnueabihf/4.9.3

## Example
### cross compile boost
 
	download newest boost library(boost_1_66_0.tar.bz2)
	tar --bzip2 -xf /path/to/boost_1_66_0.tar.bz2
	sudo su -m
	./bootstrap.sh --prefix=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf	/sysroot/usr/local
	nano ~/user-config.jam
        using gcc : arm : arm-linux-gnueabihf-g++ ; 
### simple version

	./b2 --with-atomic --with-chrono --with-date_time --with-filesystem --with-regex --with-serialization --with-thread 	--with-system --no-samples --no-tests toolset=gcc-arm link=static cxxflags=-fPIC install

### install all 
	./b2 toolset=gcc-arm install
	exit 
	rm ~/user-config.jam

### cross compile OpenSSL 1.0.2n

	Download OpenSSL and extract it
    	Version 1.0.x is working.
    	Version 1.1.x is not working well, not use !!!
	wget https://www.openssl.org/source/openssl-1.0.2l.tar.gz
	tar xzf openssl-1.0.2n.tar.gz
	cd openssl-1.0.2n
	./Configure --prefix=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf		/sysroot/usr/local os/compiler:arm-linux-gnueabihf
	make CC="arm-linux-gnueabihf-gcc" AR="arm-linux-gnueabihf-ar r" RANLIB="arm-linux-gnueabihf-ranlib"
	sudo make install

### cross compile cpp-netlib

	download cpp-netlib 0.12.0-final.tar.gz (cpp-netib 0.13.0 is not stable now, not use!) 
	tar xzf cpp-netlib 0.12.0-final.tar.gz 
	cd cpp-netlib 
	mkdir build 
	cd build 
	cmake -DCMAKE_TOOLCHAIN_FILE=$HOME/raspberrypi/pi.cmake -DOPENSSL_ROOT_DIR=/home/sammysun/raspberrypi/tools/arm-bcm270/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/local/ssl -DBOOST_ROOT_DIR=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/local/include -DCMAKE_SKIP_BUILD_RPATH=FALSE -DCMAKE_BUILD_WITH_INSTALL_RPATH=FALSE -DCMAKE_INSTALL_PREFIX:PATH=/home/sammysun/raspberrypi/tools/arm-bcm2708/arm-rpi-4.9.3-linux-gnueabihf/arm-linux-gnueabihf/sysroot/usr/local -DCMAKE_INSTALL_RPATH_USE_LINK_PATH=TRUE ../cpp-netlib 
	make
	sudo make install

### cross compile PiNet

	cd ~/PiNet
	mkdir raspberrypi_build
	cd raspberrypi_build
	cmake -DCMAKE_TOOLCHAIN_FILE=$HOME/raspberrypi/pi.cmake  ../src 
	make
	P.s. if we get error wenn cross compile PiNet cpp-netlib/include/boost/network/uri/builder.hpp:9:10: fatal error: 	'asio/ip/address.hpp' file not found
	Manually copying $SRCDIR/deps/asio/asio/include/asio to $CMAKE_INSTALL_PREFIX/include/asio fixes the problem. 
	

## secure copy built PiNet to raspberrypi 

	scp PiNet pi@10.42.0.212:/home/pi/PiNet/build
	(default password:raspberry)
## test output of PiNet
	ssh pi@10.42.0.212 ./PiNet/build/PiNet

## more detail information see reference:
[1] https://github.com/raspberrypi/tools <br />
[2] https://github.com/Yadoms/yadoms/wiki/Cross-compile-for-raspberry-PI <br />
[3] http://blog.shahada.abubakar.net/post/raspberry-pi-software-development-with-a-cross-compiler <br />
[4] https://github.com/raspberrypi/tools/issues/50 <br />
[5] https://medium.com/@au42/the-useful-raspberrypi-cross-compile-guide-ea56054de187 <br />
[6] https://stackoverflow.com/questions/21113434/error-cross-compiling-cpp-netlib-for-raspbian <br />
[7] https://stackoverflow.com/questions/16040128/hook-up-raspberry-pi-via-ethernet-to-laptop-without-router <br />
