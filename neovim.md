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


## Ubuntu 20.04 on WSL
Build and install
```
$ sudo -i apt install ninja-build gettext libtool libtool-bin autoconf automake cmake g++ pkg-config unzip
$ git clone https://github.com/neovim/neovim.git
$ cd neovim
$ git checkout stable
$ make CMAKE_BUILD_TYPE=Release
$ sudo make install
```

Share clipboard with windows (Ref: ttps://github.com/neovim/neovim/wiki/FAQ#how-to-use-the-windows-clipboard-from-wsl)
```
$ curl -sLo/tmp/win32yank.zip https://github.com/equalsraf/win32yank/releases/download/v0.0.4/win32yank-x64.zip
$ unzip -p /tmp/win32yank.zip win32yank.exe > /tmp/win32yank.exe
$ chmod +x /tmp/win32yank.exe
$ mv /tmp/win32yank.exe /usr/local/bin/
```

win32yank.exeの実行にvcruntime140.dllが必要なためインストール 
(参考: https://www.javadrive.jp/php/install/index2.html#section1)

win32yank.exeの動作確認
```
$ win32yank.exe
Invalid arguments.

Usage:
    win32yank -o [--lf]
    win32yank -i [--crlf]
```
