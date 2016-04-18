#Elasticsearch + WooCommerce

##Elasticsearch Serviceとは
オープンソースの全文検索エンジンシステムであるElasticsearchが利用できるマネージドサービスです。
Amazon RDSのように簡単により強力な検索システムを利用することができます。

##Elasticsearch Serviceを立ち上げる
- AWS管理画面へログイン
- Elasticsearch Serviceをクリック
- 「Get started」をクリック

###Step1: ドメインの設定
- 「Elasticsearch domain name」に任意のドメイン名を入力

###Step2:クラスターの設定

|設定項目|設定値|
|Instance Type|t2.micro|
|Instance count| 1|
|Storage Type|EBS|
|EBS volume type|General Purpose(SSD)|

他はデフォルトでOKです。

###Step3:アクセスポリシーの設定
- 「Select a template」をクリック
- 「Allow open access to the domain」を選択  
実運用時はIPなどでの制限をお勧めします

###Final:立ち上げ
- セットアップ完了まで10~20分ほどかかります。
- 「Status: Green」になっていれば完了です。

##Elasticommerceのセットアップ
- WordPress管理画面にログイン
- プラグインの新規追加で以下の２プラグインをインストールと有効化
 - Elasticommerce Search Form
 - Elasticommerce Related Items
- 必要な設定を入力
