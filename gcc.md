# gcc build

## バージョン
9.2.0

## Ubuntu 18.04 on WSL
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
