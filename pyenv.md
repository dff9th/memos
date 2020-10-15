# pyenv/virtualenv installメモ

## Ubuntu 18.04 on WSL
```
$ sudo -i apt install zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev libssl-dev libffi-dev
$ git clone https://github.com/pyenv/pyenv.git $XDG_CACHE_HOME/pyenv
$ git clone https://github.com/yyuu/pyenv-virtualenv.git $XDG_CACHE_HOME/pyenv/plugins/pyenv-virtualenv
$ vi ~/.bashrc
# pyenv
if [ -d $XDG_CACHE_HOME/pyenv ]; then
    export PYENV_ROOT="$XDG_CACHE_HOME/pyenv"
    export PATH="$PYENV_ROOT/bin:$PATH"
    eval "$(pyenv init - --no-rehash)"
    if [ -d $PYENV_ROOT/plugins/pyenv-virtualenv ]; then
        eval "$(pyenv virtualenv-init - --norehash)"
    fi
fi
$ exec $SHELL
$ pyenv install 3.8.2
$ pyenv install 2.7.17
```

## CentOS 7
```
$ sudo -i yum install gcc zlib-devel bzip2 bzip2-devel readline readline-devel sqlite sqlite-devel openssl openssl-devel git libffi-devel
$ git clone https://github.com/pyenv/pyenv.git $XDG_CACHE_HOME/pyenv
$ git clone https://github.com/yyuu/pyenv-virtualenv.git $XDG_CACHE_HOME/pyenv/plugins/pyenv-virtualenv
# pyenv
if [ -d $XDG_CACHE_HOME/pyenv ]; then
    export PYENV_ROOT="$XDG_CACHE_HOME/pyenv"
    export PATH="$PYENV_ROOT/bin:$PATH"
    eval "$(pyenv init - --no-rehash)"
    if [ -d $PYENV_ROOT/plugins/pyenv-virtualenv ]; then
        eval "$(pyenv virtualenv-init - --norehash)"
    fi
fi
$ exec $SHELL
$ pyenv install 3.8.2
$ pyenv install 2.7.17
```
