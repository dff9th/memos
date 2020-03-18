# git build

## バージョン
2.25.2

## Ubuntu 18.04 on WSL
```
$ sudo -i apt install asciidoc xmlto docbook2x
$ curl -LO https://mirrors.edge.kernel.org/pub/software/scm/git/git-2.25.2.tar.gz
$ tar zxf git-2.25.2.tar.gz
$ cd git-2.25.2
$ make configure
$ ./configure --prefix=/usr/local
$ make all doc info
$ sudo make install install-doc install-html install-info
$ sudo mkdir /usr/local/share/completion
$ sudo cp ./contrib/completion/git-completion.* /usr/local/share/completion/
```
