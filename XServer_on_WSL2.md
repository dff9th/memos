# WSL2経由のX Server構築


## 参考
- https://orebibou.com/2019/02/wsl%E3%81%8B%E3%82%89ssh%E6%8E%A5%E7%B6%9A%E5%85%88%E3%81%AE%E5%87%BA%E5%8A%9B%E7%B5%90%E6%9E%9C%E3%82%92%E3%82%AF%E3%83%AA%E3%83%83%E3%83%97%E3%83%9C%E3%83%BC%E3%83%89%E3%81%AB%E3%82%B3%E3%83%94/
- https://qiita.com/ryoi084/items/0dff11134592d0bb895c
- https://github.com/microsoft/WSL/issues/4106


## X server (Windows10ホストPC) 側の作業
- VcXrvのインストール (https://sourceforge.net/projects/vcxsrv/)
- XLaunch実行
    - Extra settingsでNative openglを無効，Additional parameters for VcXsrvに"-ac"
- WSLからのアクセスに対するfirewallの無効化 (Powershell管理者権限で実行)
  ```
  Set-NetFirewallProfile -Name public -DisabledInterfaceAliases "vEthernet (WSL)"
  ```

## WSL2側の作業
xeyesで疎通確認 (実行時に目玉のGUIソフトが立ち上がればOK)
```
$ sudo apt install x11-apps
$ echo $(netsh.exe interface ip show addresses "vEthernet (WSL)" | grep "IP Address:" | awk 'BEGIN{RS="\r\n"}{print $3}'
172.29.48.1
$ export DISPLAY=172.29.48.1:0.0
$ xeyes
```
