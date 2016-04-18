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
- WooCommerce Powered by AMIMOTO (HHVM) on AWS Marketplace : https://aws.amazon.com/marketplace/pp/B00ZGTRMVU/
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


##AMIMOTOにS3を追加する
