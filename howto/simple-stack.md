#JINKEI Simple Stackを作る
![]()
作る構成図

##この構成図でできること
- RDSを使うことで耐障害性を向上
- CloudFrontでコンテンツの配信を高速化
- S3を使った低コストメディアストレージ

##事前準備
- AMIMOTO AMIでEC2をセットアップしてください
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

###AWS CLIを動かすためのIAM設定
AWS CLIを動かすにはクレデンシャル（認証情報）が必要です。
ここからはIAM (AWS Identity and Access Management) を使ってAWS CLIのための認証情報を取得します。

####AWSコンソールからIAMのページへ移動
https://console.aws.amazon.com/iam/home#home

####ユーザーを作成
- 左メニューの「Users」をクリック
- 「Create News Users」をクリックしてウィザードを起動
- 「Enter User names:」に「awscli」と入力  
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
aws configure --profile amimoto
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

####AWS CLIの動作確認
以下のコマンドでエラーが出なければ設定完了です。
```
$ aws s3 ls
```

###CloudFrontをAWS CLIから使用する準備
通常のAWS CLIではCloudFrontが利用できないため、有効化させます
```
$ aws --profile amimoto  configure set preview.cloudfront true
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

```
$ export origin_url='{ORIGIN URL}'; export domain='{ORIGIN DOMAIN NAME HERE}'; aws cloudfront create-distribution --cli-input-json "$(curl -l -s https://raw.githubusercontent.com/amimoto-ami/create-cf-dist-settings/master/source_dist_setting.sh | sh)"
```

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
- Endpointのアドレス右側に「No Inboud Permission」という赤文字がある場合、セキュリティグループの設定を変更します
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
$ mysql -h {RDS_ENDPOINT} -u {Master Username} -p{Master Password} {Database Name} < /tmp/dump.sql
```

####余談：本番環境の移行に自信がない方は・・・
AWS Database Migration Service:https://aws.amazon.com/jp/dms/

###AMIMOTOのDBをRDSにつなぎかえる
####wp-config.phpを編集
```
$ cd /var/www/vhosts/{INSTANCE_ID}
$ vim wp-config.php
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



##AMIMOTOにS3を追加する
