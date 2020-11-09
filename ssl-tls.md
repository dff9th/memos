# SSL/TLSの仕組み

## 参考
- https://christina04.hatenablog.com/entry/ssl-tls-flow
- https://www.kakiro-web.com/linux/ssl-client.html


## 用語
- 証明書（.crt）
- 証明書要求（.csr）
- 署名


## 証明書とは
構成
- 発行者/認証局（Issuer）
- 発行先/認証対象（Subject）
- 公開鍵
- 認証局による署名


## 証明書要求とは
構成
- 発行者/認証局（Issuer）
- 認証局の公開鍵


## 署名とは
署名を除いた証明書の内容から生成したハッシュ値を，認証局の秘密鍵で暗号化（RSA署名）したもの


## 証明書に含まれる署名の検証（サーバ認証）
1. 証明書から証明書要求を作成
2. 認証局に証明書要求を送り，認証局の証明書を取得
3. 認証局の証明書から認証局の公開鍵を抽出
4. 署名を認証局の公開鍵で復号し，証明書のハッシュ値を取得
5. 署名を除いた証明書の内容からハッシュ値を生成
6. 4と5のハッシュ値が同一か検証
7. 1から6をクライアント環境内に含まれるルート証明書に到達するまで繰り返す


## 自己認証局の準備手順 (CentOS7)
ルート証明書作成
```bash
# openssl req -x509 -newkey rsa:2048 -nodes -keyout myca.key -sha256 -days 3650 -out myca.crt
Generating a 2048 bit RSA private key
...................+++
...+++
writing new private key to 'myca.key'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:JP
State or Province Name (full name) []:MyCaState
Locality Name (eg, city) [Default City]:MyCaLocality
Organization Name (eg, company) [Default Company Ltd]:MyCaOrganization
Organizational Unit Name (eg, section) []:MyCaUnit
Common Name (eg, your name or your server's hostname) []:myca.gcp.local
Email Address []:
```

署名用のインデックスファイルとシリアル番号生成
```bash
# touch /etc/pki/CA/index.txt
# echo 01 > /etc/pki/CA/serial
```


## サーバ証明書の作成および自己認証局による署名
例としてgitlab.gcp.localというドメイン用のサーバ証明書を作成

秘密鍵生成
```bash
# openssl genrsa -out gitlab.gcp.local.key 2048
Generating RSA private key, 2048 bit long modulus
..............................................................+++
..........................+++
```

証明書要求生成
```bash
# openssl req -new -key gitlab.gcp.local.key -out gitlab.gcp.local.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [XX]:JP
State or Province Name (full name) []:MyCaState
Locality Name (eg, city) [Default City]:MyCaLocality
Organization Name (eg, company) [Default Company Ltd]:MyCaOrganization
Organizational Unit Name (eg, section) []:MyCaUnit
Common Name (eg, your name or your server's hostname) []:gitlab.gcp.local
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
```

証明書生成
```bash
# openssl ca -in gitlab.gcp.local.csr -out gitlab.gcp.local.crt -keyfile myca.key -cert myca.crt -md sha256 -days 3650
Using configuration from /etc/pki/tls/openssl.cnf
Check that the request matches the signature
Signature ok
The stateOrProvinceName field needed to be the same in the
CA certificate (MyCaState) and the request (MyGitlabState)
```


## 自己認証局のルート証明書登録
```
# cp myca.crt /etc/pki/ca-trust/source/anchors/
# update-ca-trust
```
