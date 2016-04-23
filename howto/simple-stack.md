#JINKEI Simple Stackを作る
![](./img/stack.png)
作る構成図

##この構成図でできること
- RDSを使うことで耐障害性を向上
- CloudFrontでコンテンツの配信を高速化
- S3を使った低コストメディアストレージ

##事前準備
- AMIMOTO AMIでEC2をセットアップしてください(WordPressのインストールまで必要です)
- AWS CLIを使用しますので、これからセットアップします
- CloudFrontをAWS CLIから使用する準備も必要です

###AMIMOTO AMIでEC2をセットアップしてください
- ハンズオン最中にDBの切り替えなどを行いますので、本番環境は絶対に使わないでください
- WooCommerce Powered by AMIMOTO (Apache HTTPD PHP7) on AWS Marketplace : https://aws.amazon.com/marketplace/pp/B01DAONMCK/
- AMI料金は14日間無料（EC2インスタンス料金は必要）です

###AWS CLIのセットアップ
aws-cli のインストール方法は二通り:

- AWS ユーザガイドページの手順で行う:  
http://docs.aws.amazon.com/cli/latest/userguide/installing.html
- Mac の場合は パッケージマネージャの Homebrew を使ってインストールをする:  
http://brew.sh/index.html

####Homebrewからセットアップする
以下のコマンドをターミナルへ入力します

- /usr/bin/ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"
- brew install wget
- brew install awscli

#### Windowsにインストールする
- Download the AWS CLI MSI installer for Windows (64-bit):https://s3.amazonaws.com/aws-cli/AWSCLI64.msi
- Download the AWS CLI MSI installer for Windows (32-bit):https://s3.amazonaws.com/aws-cli/AWSCLI32.msi

デフォルトでは以下のディレクトリに展開されます

- C:\Program Files\Amazon\AWSCLI (64-bit)
- C:\Program Files (x86)\Amazon\AWSCLI (32-bit)

###AWS CLIを動かすためのIAM設定
AWS CLIを動かすにはクレデンシャル（認証情報）が必要です。
ここからはIAM (AWS Identity and Access Management) を使ってAWS CLIのための認証情報を取得します。

####AWSコンソールからIAMのページへ移動
https://console.aws.amazon.com/iam/home#home

####ユーザーを作成
- 左メニューの「Users」をクリック
- 「Create News Users」をクリックしてウィザードを起動
- 「Enter User names:」に「amimoto-cli」と入力  
AWS CLIのためのIAMユーザであることをわかるようにしましょう
- 「Generate an access key for each user」のチェックをオンにする
- 作成します

####クレデンシャルを保存
- Access Key ID
- Secret Access Key

この２つを手元に控えてください。（CSVでダウンロード可能）
この画面を閉じると２度と見れませんので、紛失した場合は再発行となります。

####AWS CLIの初期設定
```
aws configure --profile amimoto-cli
```

#####設定する値
以下の値を対話式で入力します

- クレデンシャル
- デフォルトリージョン
- 実行結果を表示するフォーマット

```
AWS Access Key ID [None]: xxxxxxxxxx
AWS Secret Access Key [None]: xxxxxxxxxx
Default region name [None]: ap-northeast-1
Default output format [None]: json
```

###CloudFrontをAWS CLIから使用する準備
通常のAWS CLIではCloudFrontが利用できないため、有効化させます
```
$ aws --profile amimoto-cli configure set preview.cloudfront true
##動作確認
$ aws --profile amimoto-cli  cloudfront help
```

##AMIMOTOにCloudFrontを追加する
CloudFrontを使うことで・・・

- 世界各地にあるAWSのもつCDNサーバを使える
- CDNを用いることでサイトの高速化とサーバ負荷削減を同時実現
- エラーレスポンスのカスタマイズが可能なので、サーバ障害時にも強くなる

###セットアップコマンド

- ORIGIN URLをAMIMOTOサーバのドメイン名（パブリックDNS）に書き換えます。
- ORIGIN DOMAIN NAME HEREに公開予定のサイトドメインを入力します。  
(ドメインを設定しない場合はORIGIN URL）を同じ値を入れてください
- profile名を「amimoto-cli」以外にしている方は、「--profile amimoto-cli」の部分を変更する必要があります。
```
$ export origin_url='{ORIGIN URL}'; export domain='{ORIGIN DOMAIN NAME HERE}'; aws --profile amimoto-cli  cloudfront create-distribution --cli-input-json "$(curl -l -s https://raw.githubusercontent.com/amimoto-ami/create-cf-dist-settings/master/source_dist_setting.sh | sh)"
```
####パブリックDNSがない場合
http://qiita.com/kasokai/items/4ea689ce9f206e78a523

###セットアップを待ちます
CloudFrontが立ち上がるまで２０〜３０分程度かかります。
待機している間にRDSとS3のセットアップを進めましょう。

##AMIMOTOにRDSを追加する
RDSを使用することで・・・

- 管理の簡単なDBサーバを簡単に使える
- DBのレプリケーションやスペック変更が１クリックで実現可能
- MySQL / MariaDB / Amazon Auroraなど、様々なエンジンを使える

###RDSをセットアップする
- AWS管理画面へアクセス
- RDSのアイコンをクリック
- 「Instance」から「Launch DB Instance」をクリック
- DBエンジンを選択（MariaDBを推奨）
- Amazon Auroraを勧められますが、お金に余裕がないなら避けましょう

###RDSの初期設定
「Specify DB Details」という画面でDBの初期設定を行います

####「Settings」に入れる値

|項目名|入れる値|
|:--|:--|
|DB Instance Identifier|DBインスタンス名|
|Master Username|DBのルートユーザー名|
|Master Password|DBのルートユーザーパスワード|
|Confirm Passoword|パスワードの確認|

DBに接続する際に必須の値ですので、手元にメモしておきましょう。
その他の設定は（ハンズオンでは）特に変更しなくてOKです。

####「Configure Advanced Settings」でDB名を設定する
「Database Name」という項目に入れた値がDB名になりますので、これも控えておきましょう。

#####こんなイメージです
```
$ mysql -u {Master Username} -p{Master Password} {Database Name}
```

####RDSを作成します
作成には少し時間がかかります。
今の間に一息いれましょう。

左メニュー「Instance」をクリックして「DB Instance Identifier」と同じ名前の「DB Instance」を探してください。
一致するものの「Status」が「available」になっていればOKです。

####セキュリティグループを設定
- セキュリティグループの設定を変更します
- 「Edit Security Group」をクリックする
- 「Inboud」タブをクリックする
- 「Add Rule」をクリックする
- 「Type」を「MySQL/Aurora」に設定する
- 「Source」は「Anywhere」を選択する  
可能な方はAMIMOTO AMIのIP（XX.XXX.XXX.XX/0）にしてみてください
- 「Save」をクリック

###AMIMOTOのDB情報をRDSに移行する
####AMIMOTOにSSH接続
AMIMOTOのインスタンスにSSHで接続してください。
```
$ ssh -i /path/to/pem/{PEMFILENAME}.pem ec2-user@{INSTANCE_IP}
```
####WP-CLIからAMIMOTOのDB情報をエクスポート
```
$ cd /var/www/vhosts/{INSTANCE_ID}
$ wp db export /tmp/dump.sql
```
####MySQLでRDSにデータをインポート
先ほどメモした値を使います
```
$ mysql -h {RDS_ENDPOINT} -u {Master Username} -p {Database Name} < /tmp/dump.sql
```
*パスワードを要求されますので、{Master Password}を入力します。
*{RDS_ENDPOINT}には「:3306」というポート番号は不要です。

###AMIMOTOのDBをRDSにつなぎかえる
####wp-config.phpを編集
```
$ sudo su -
# cd /var/www/vhosts/{INSTANCE_ID}
# vim wp-config.php
```

####書き換える場所
wp-config.php(Line:26-33)
```
if ( !$db_data ) {
    $db_data = array(
        'database' => '{Database Name}',
        'username' => '{Master Username}',
        'password' => '{Master Password}',
        'host'     => '{RDS_ENDPOINT}',
    );
}
```
#####vimを使う場合・・・

|コマンド|できること|
|:--|:--|
|[esc]|ノーマルモード|
|:set number|行数表示|
|[shift]+[z]２回|保存して終了|
|i|編集モード|

####接続を確認
AMIMOTOのサイトにアクセスして、「データベース接続エラー」が表示されていないことを確認します。

####EC2のMySQLを停止する
```
# exit
$ vim /opt/local/amimoto.json
```
以下のように「"mysql": { "enabled": false },」を追加します。
#####Before
```
{
	"mod_php7" : { "enabled": true },
	"run_list" : [ "recipe[amimoto]" ]
}
```

######After
```
{
	"mod_php7" : { "enabled": true },
	"mysql": { "enabled": false },
	"run_list" : [ "recipe[amimoto]" ]
}
```

#####Run Command
```
$ sudo /opt/local/provision
$ sudo service mysql stop
```

##AMIMOTOにS3を追加する
###S3を使うメリット
- 低コストでメディアストレージを使える
- 冗長化されて保存されるので、障害に強い
- ファイル数・容量に上限なし

###セットアップ手順
- IAMを準備
- バケットを作成
- バケットの設定
- WordPressプラグインのセットアップ

####IAMを準備
#####IAMユーザーを作成
- 管理画面からIAMにアクセス
- 左メニューの「Users」をクリック
- 「Create News Users」をクリックしてウィザードを起動
- 「Enter User names:」に「amimoto-s3」と入力  
S3のためのIAMユーザであることをわかるようにしましょう
- 「Generate an access key for each user」のチェックをオンにする
- 作成します

#####IAMユーザーにポリシーを設定
- 左メニューの「Users」をクリック
- 「s3amimoto」をクリック
- 「Permissions」をクリック
- 「Managed Policies」の枠内にある「Attach Policy」をクリック
- 「AmazonS3FullAccess」を選択して「Attach Policy」をクリック

####バケットを作成
- 管理画面からS3にアクセス
- 「Create Bucket」をクリック
- 「Bucket Name」に任意の名前をいれます  
Note:すでに誰かが使っているバケット名は利用できません
- 「Region」を選択します
- 「Create」をクリックして作成します

####バケットの設定
- S3のリストから先ほど作成したバケット名を選択します
- 画面右上「Properties」をクリックします
- 「Static Website Hosting」をクリックします
- 「Endpoint」を控えます
- 「Enable website hosting」を選択します
- 「Index Document」に「index.html」を入力します
- 「Save」を選択します

####WordPressプラグインを入れる
- WordPress管理画面にログインします
- Nephila Clavata(絡新婦)プラグインを有効化します
- 「Settings > Nephila Clavata」からS3の設定を行います

|項目名|入れる値|
|:--|:--|
|AWS Access Key|amimoto-s3のAWS Access Key|
|AWS Secret Key|amimoto-s3のAWS Secret Key|
|AWS Region|S3バケット作成時に指定したリージョン|
|S3 Bucket|作成したS3バケットの名前|
|S3 URL|S3バケットの「Endpoint」|
|Storage Class|STANDARD|

##CloudFrontの最終設定
最後にCloudFrontをWordPressで便利に使う設定を行います。

###設定手順
- IAMを準備
- WordPressプラグインのセットアップ

####IAMを準備
#####IAMユーザーを作成
- 管理画面からIAMにアクセス
- 左メニューの「Users」をクリック
- 「Create News Users」をクリックしてウィザードを起動
- 「Enter User names:」に「amimoto-cloudfront」と入力  
CloudFrontのためのIAMユーザであることをわかるようにしましょう
- 「Generate an access key for each user」のチェックをオンにする
- 作成します

#####IAMユーザーにポリシーを設定
- 左メニューの「Users」をクリック
- 「amimoto-cloudfront」をクリック
- 「Permissions」をクリック
- 「Managed Policies」の枠内にある「Attach Policy」をクリック
- 「CloudFrontFullAccess」を選択して「Attach Policy」をクリック

####WordPressプラグインのセットアップ
#####キャッシュ削除プラグイン
- WordPress管理画面にログインします
- C3 CloudFront Clear Cacheプラグインを有効化します
- 「Settings > C3 Settings」からCloudFrontの設定を行います

|項目名|入れる値|
|:--|:--|
|CloudFront Distribution ID|CloudFrontのディストリビューションID|
|AWS Access Key|amimoto-cloudfrontのAWS Access Key|
|AWS Secret Key|amimoto-cloudfrontのAWS Secret Key|

#####CloudFrontのディストリビューションID確認方法
![](./img/cf_distrib.png)

#####プレビューの更新を反映するためのプラグイン
```
$ cd /var/www/vhost/{INSTANCE_ID}/wp-content
$ mkdir mu-plugins && cd mu-plugins
$ wget wget https://gist.githubusercontent.com/wokamoto/ecfd3a7ea9ef80ea1628/raw/02e4e011597c0969f0ff4dec48de539a89b96e4a/cloudfront-preview-fix.php
```
