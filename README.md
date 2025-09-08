AWS Client VPNの設定
===

## タスク1: VPCの構築

### 1. cloudformation/vpc.ymlを使ってCloudFormationでVPCを構築します。

## タスク2: 証明書の作成

開発コンテナ (DevContainer) で証明書を作成します。

### 2. GitHubからOpenVPN/easy-rsaをクローンします。

```
git clone https://github.com/OpenVPN/easy-rsa.git
```

### 3. 認証機関を初期化し作成します。

```
cd easy-rsa/easyrsa3
./easyrsa init-pki
./easyrsa build-ca nopass
```

### 4. サーバー証明書を作成します。

```
./easyrsa --san=DNS:server build-server-full server nopass
```

### 5. クライアント証明書を作成します。

```
./easyrsa build-client-full client.domain.tld nopass
```

### 6. キーと証明書をコピーします。

```
cd ../..
mkdir ssl
cp easy-rsa/easyrsa3/pki/ca.crt ssl
cp easy-rsa/easyrsa3/pki/issued/server.crt ssl
cp easy-rsa/easyrsa3/pki/private/server.key ssl
cp easy-rsa/easyrsa3/pki/issued/client.domain.tld.crt ssl
cp easy-rsa/easyrsa3/pki/private/client.domain.tld.key ssl
```

### 7. AWSマネジメントコンソールでACMにサーバー証明書をインポートします。

AWS Certificate Managerを開いて、証明書を一覧から、インポートを選択します。

7-1. 証明書の詳細の`証明書本文`にssl/server.crtの内容をコピーします。

7-2. 証明書の詳細の`証明書のプライベートキー`にssl/server.keyの内容をコピーします。

7-3. 証明書の詳細の`証明書チェーン - オプション`にssl/ca.crtの内容をコピーします。

7-4. `証明書をインポート`のボタンをクリックします。

### 8. AWSマネジメントコンソールでACMにサーバー証明書をインポートします。

AWS Certificate Managerを開いて、証明書を一覧から、インポートを選択します。

8-1. 証明書の詳細の`証明書本文`にssl/client.domain.tld.crtの内容をコピーします。

8-2. 証明書の詳細の`証明書のプライベートキー`にssl/client.domain.tld.keyの内容をコピーします。

8-3. 証明書の詳細の`証明書チェーン - オプション`にssl/ca.crtの内容をコピーします。

8-4. `証明書をインポート`のボタンをクリックします。

### 9. クライアント VPN エンドポイントを構築します。

AWSマネジメントコンソールでVPCを開きます。

9-1. クライアント VPN エンドポイントを選択します。

9-2. `クライアント VPN エンドポイントを作成`のボタンをクリックします。

9-3. 名前タグ - オプションに`client-vpn-endpoint`を入力します。

9-4. Endpoint IP address typeは、`Dual stack`を選択します。

9-5. Traffic IP address typeは、`IPv4`を選択します。

9-6. クライアント IPv4 CIDRに`10.0.16.0/20`を入力します。


9-7. 認証情報のサーバー証明書 ARNに`server`のARNを入力します。

9-8. 認証情報の認証オプションは、`相互認証`を選択します。

9-9. 認証情報のクライアント証明書 ARNに`client.domain.tld`のARNを入力します。


9-10. DNS configuration - オプションの`DNS サーバー 1 IP アドレス`に`10.0.0.2`を入力します。


9-11. その他のパラメータ - オプションの`スプリットトンネルを有効化`をオンにします。

9-12. その他のパラメータ - オプションのVPC IDにCloudFormationで作成したVPCを入力します。


9-13. `クライアント VPN エンドポイントを作成`のボタンをクリックします。

### 10. クライアント VPN エンドポイントをターゲットネットワークに関連付ける

クライアント VPN エンドポイントで作成した`client-vpn-endpoint`を選択します。

10-1. `ターゲットネットワークの関連付け`のタブを選択します。

10-2. `ターゲットネットワークに関連付ける`ボタンをクリックします。

10-3. VPCにCloudFormationで作成したVPCを選択します。

10-4. 関連付けるサブネットを選択に`PrivateSubnet1`を選択します。

10-5. `ターゲットネットワークに関連付ける`ボタンをクリックします。

### 11. クライアント VPN エンドポイントに承認ルールを作成します。

クライアント VPN エンドポイントで作成した`client-vpn-endpoint`を選択します。

11-1. `承認ルール`のタブを選択します。

11-2. `承認ルールを作成`ボタンをクリックします。

11-3. アクセスを有効にする送信先ネットワークに`10.0.0.0/16`を入力します。

11-4. `認証ルールを追加`ボタンをクリックします。

### 12. クライアント設定

クライアント VPN エンドポイントで作成した`client-vpn-endpoint`を選択します。

12-1. `クライアント設定をダウンロード`ボタンをクリックします。

12-2. ダウンロードしたファイルの`</ca>`の後に、ssl/client.domain.tld.crtの内容を`<cert>`と`</cert>`の間に入力します。

12-3. `</cert>`の後に、ssl/client.domain.tld.keyの内容を`<key>`と`</key>`の間に入力します。
