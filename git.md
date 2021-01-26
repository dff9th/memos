# git build

## CentOS 7
2.26.2
```
$ sudo -i yum install asciidoc xmlto docbook2X docbook-utils
$ sudo ln -s /bin/db2x_docbook2texi /bin/docbook2x-texi
$ cd ~/tmp
$ git clone https://github.com/git/git.git
$ cd git
$ git checkout v2.26.2
$ make configure
$ ./configure --prefix=/usr/local
$ make all doc info
$ sudo make install install-doc install-html install-info
$ sudo mkdir /usr/local/share/completion
$ sudo cp ./contrib/completion/git-completion.* /usr/local/share/completion/
```
参考
- https://www.server-memo.net/memo/github/github-install.html

## Ubuntu 20.04 on WSL
2.30.0
```
$ sudo -i apt install asciidoc xmlto docbook2x libcurl4-openssl-dev
$ cd ~/tmp
$ git clone https://github.com/git/git.git
$ cd git
$ git checkout v2.30.0
$ make configure
$ ./configure --prefix=/usr/local
$ make all doc info
$ sudo make install install-doc install-html install-info
$ sudo mkdir /usr/local/share/completion
$ sudo cp ./contrib/completion/git-completion.* /usr/local/share/completion/
```
