# Neovim Build

## CentOS 7
```
$ sudo -i yum -y install ninja-build libtool autoconf automake cmake gcc gcc-c++ make pkgconfig unzip patch
$ git clone https://github.com/neovim/neovim.git
$ cd neovim
$ git checkout stable
$ make CMAKE_BUILD_TYPE=Release
$ sudo make install
```

## Ubuntu 18.04 on WSL
```
$ sudo -i apt install ninja-build gettext libtool libtool-bin autoconf automake cmake g++ pkg-config unzip
$ git clone https://github.com/neovim/neovim.git
$ cd neovim
$ git checkout stable
$ make CMAKE_BUILD_TYPE=Release
$ sudo make install
```
