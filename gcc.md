# gcc build

## CentOS 7 
9.3.0
```
$ curl -LO http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/gcc-9.2.0/gcc-9.2.0.tar.gz
$ tar zxf gcc-9.2.0.tar.gz
$ cd gcc-9.2.0
$ ./contrib/download_prerequisites
$ mkdir build
$ cd build
$ ../configure --enable-languages=c,c++ --prefix=/usr/local --disable-bootstrap --disable-multilib
$ make > /dev/null
$ sudo make install > /dev/null
$ sudo vi /etc/ld.so.conf.d/local_lib64.conf
/usr/local/lib64
$ sudo ldconfig
```

## Ubuntu 18.04 on WSL
9.2.0
```
$ curl -LO http://ftp.tsukuba.wide.ad.jp/software/gcc/releases/gcc-9.2.0/gcc-9.2.0.tar.gz
$ tar zxf gcc-9.2.0.tar.gz
$ cd gcc-9.2.0
$ ./contrib/download_prerequisites
$ mkdir build
$ cd build
$ ../configure --enable-languages=c,c++ --prefix=/usr/local --disable-bootstrap --disable-multilib
$ make > /dev/null
$ sudo make install > /dev/null
$ sudo vi /etc/ld.so.conf.d/local_lib64.conf
/usr/local/lib64
$ sudo ldconfig
```
