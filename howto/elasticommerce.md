#Elasticsearch + WooCommerce

##Elasticsearch Serviceとは
オープンソースの全文検索エンジンシステムであるElasticsearchが利用できるマネージドサービスです。
Amazon RDSのように簡単により強力な検索システムを利用することができます。

##事前準備
- AMIMOTO AMIを起動させてください
- WooCommerceプラグインのインストールと有効化、初期設定を完了させてください

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

###Elasticommerce Search Formの設定
- 「Settings > Elasticommerce Service」を選択
- 「Elasticsearch Endpoint」に作成したElasticsearchのドメインを入力して保存します

###Elasticommerce Related Itemsの設定
- 「Settings > Elasticommerce Service」を選択
- 以下の設定が可能です

|項目名|説明|
|:--|:--|
|Search Score|検索結果の関連性をフィルタする（数値が高いほど厳密に）|
|Select Search Target|関連性検索を行う対象を指定する|

- 「Import All Products」をクリックすることで、手動でWooCommerceの商品をElasticsearchにインポートできます

###Elasticommerce Search Formを使う
WooCommerceの標準機能である商品検索機能を、自動でElasticsearchの検索結果に置き換えます。

###Elasticommerce Related Itemsを使う
WooCommerceの標準機能である関連商品機能を、自動でElasticsearchの検索結果に置き換えます。  
また関連商品表示ウィジェットとして「Elasticommerce Related Widgets」が利用できます。  
さらに専用の関数を使うことで任意の処理を作成できます。

|関数名|動き|
|:--|:--|
|escr_related_item();|表示しているページと関連性の高い商品を表示する|
|escr_get_related_item();|表示しているページと関連性の高い商品データを取得する|
