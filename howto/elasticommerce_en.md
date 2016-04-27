# Elasticsearch + WooCommerce

## What is Elasticsearch Service
Elasticsearch Service is the managed service for who wants to use open source full text search engine Elasticsearch. You can use powerful search systems so easily like Amazon RDS.

## Preparation
- Firstly, start AMIMOTO AMI
- Secondary, install and enable WooCommerce Plugin for WordPress, then complete initial set ups.

## Start Elasticsearch Service
- Login to AWS Console
- Click Elasticsearch Service
- Click [Get started]

### Step1: Set up your domain name
- input your domain name to [Elasticsearch domain name]

### Step2: Set up Cluster

|Item|Value|
|:--|:--|
|Instance Type|t2.micro|
|Instance count|1|
|Storage Type|EBS|
|EBS volume type|General Purpose(SSD)|

Default works fine for the other items.

### Step3: Attach access policy to Elasticsearch 
- Click [Select a Templae]
- Choose [Allow open access to the domain]
Note: we recommend set IP address restrictions on production stage.

### Step4 Start up the service
- It takes 20-30 minutes to complete set up.
- Settings are completed when a status label is sat [Status: Green] 

## Set up Elasticommerce
- Login to your WordPress Dashboard
- Install and activate below two plugins on Plugin page:
-- Elasticommerce Search Form
-- Elasticommerce Related Items
- Fill in required form

### Setting up Elasticommerce Search Form
- Click [Settings] --> [Elasticommerce Service]
- Input the Elasticsearch's domain name into [Elasticsearch Endpoint]
- Click [Save]

### Setting up Elasticommerce Related Items
- Choose [Settings] --> [Elasticommcerce Service]
-Setting items are below:

|Items|Description|
|:--|:--|
|Search Score|Filter the relativity for search results (higher value for more strict relativity)|
|Select Search Target|Order target items for relativity search|

- To import items on WooCommerce manually, just click [Import All Products]

### How to enable Elasticommerce Search Form
Replace search result from standard product search of WooCommerce to Elasticsearch's  one.

### How to use Elasticommerce Related Items
It replaces the WooCommerce's default related items feature to Elasticsearch's one. And  you can use [Elasticommerce Related Widgets] for [Related items Widgets], also you can create any process by using a dedicated function. 

|Functions|Description|
|:--|:--|
|escr_related_item();|Display related items for displayed page|
|escr_get_related_item();|Get items' data on displayed page|



