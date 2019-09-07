# Google Cloud をAPIで操作する

## 目次
ハンズオン全体の所要時間<walkthrough-tutorial-duration duration="45"></walkthrough-tutorial-duration>

- Terraform利用のための準備
   - Cloud Source Repositoryからソースコードをクローンする
   - サービスアカウントの接続情報を取得する
   - Cloud Shell上へjsonファイルをアップロードする
- Terraformを利用したインスタンス作成
   - Terraformを初期化する
   - GCE作成ソースを変更する
   - インスタンスグループ作成ソースを変更する
   - LB作成ソースを変更する
   - cloudSQL作成ソースを変更する
   - Terraformを利用してGCE、GLB、SQLを設定する
- Ansible利用のための準備
   - 対象サーバにpython,python-aptをインストールする
   - 対象ホストファイルを作成
   - Ansibleモジュールからping疎通確認
- Ansibleを利用したWordpress導入
   - Ansibleを実行する
   - Wordpressの動作をブラウザ上にて確認
   - 演習リソースの削除

## はじめに

本ハンズオンでは、以下内容を想定して記載しております。
- プロジェクト名
  - handson-project
- サービスアカウント名
  - handson-wp
- サービスアカウントの権限
  - 編集者
  - Cloud SQL 管理者
  - Compute 管理者
  - Compute Load Balancer 管理者

また、TerraformとAnsibleがインストールされていることが前提条件となります。
以下記事を参考にCloud Shell上でTerraformとAnsibleを実行できるように設定をお願い致します。  
[Cloud Shell環境を使ってCloud Shellのベースイメージを変更する](https://qiita.com/ptpt-free/items/92e8e8c446183ca314d2)

## Terraform利用のための準備
所要時間<walkthrough-tutorial-duration duration="10"></walkthrough-tutorial-duration>

### Cloud Source Repositoryからソースコードをクローンする
今回のハンズオン用に用意したコードをダウンロードします。
Cloud Shell上より、以下のコマンドを入力します。ホームディレクトリにて、下記コマンドを実行します。

```bash
cd $HOME; git clone https://github.com/ptpt-free/handson-wp.git
```

## Terraform利用のための準備

### サービスアカウントの接続情報の取得

Terraformを利用する際、GCP上のリソースへアクセスするためサービスアカウントが必要となります。  
GCPのコンソール画面へアクセスし、**IAMと管理**から**サービスアカウント**を選択します。  
下のボタンを押すと、[IAMと管理]の場所を表示します。  

<walkthrough-menu-navigation sectionid="IAM_ADMIN_SECTION"></walkthrough-menu-navigation>

**handson-wp@handson-project.iam.gserviceaccount.com**の右側にある記号を選択し、[鍵を作成]をクリックします。  
キーのタイプを**JSON**と選択し、[作成]をクリックして利用しているPCに鍵を保存します。  

## Terraform利用のための準備


### Cloud Shell上へjsonファイルをアップロードする
以下の手順を用いて、ダウンロードしたjsonファイルをアップロードし、指定したディレクトリへ移動します。
1. Cloud Shell画面の右側<walkthrough-web-preview-icon></walkthrough-web-preview-icon>の**横にあるボタン**をクリックしてください。
1. `ファイルをアップロード`を選択し、先程ダウンロードしたjsonファイルをアップロードします。
1. 画面右下にアップロード状況が表示され、[アップロードが完了しました]と表示されます。
1. アップロードされたjsonファイルがホームディレクトリ($HOME)に存在するか確認します。
1. Cloud Shell上より、以下のコマンドを入力します。

```bash
mv $HOME/handson-wp-* $HOME/handson-wp/terraform/account.json
```

## Terraformを利用したインスタンス作成
所要時間<walkthrough-tutorial-duration duration="15"></walkthrough-tutorial-duration>

### Terraformを初期化する
作成するインスタンスにSSHキーを設定するため、以下手順を利用してSSHキーを取得します。
* SSHキーを生成します。
** ~/.ssh/id_rsa**に既にキーが存在する場合、新規作成時に鍵を上書きするか確認されますので、適宜ご対応ください。

```bash
ssh-keygen -t rsa
```

* SSHキー公開鍵をコピーします。
作成した鍵を確認します。  

```bash
cat ~/.ssh/id_rsa.pub
```

以下のような鍵データが表示されます。後の手順で利用するため、メモ帳に貼り付ける等して残しておいてください。
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCy+7N4kisvLVs9FcaRUhozkwxB8Ak0AlwVbw7EQTkFmWwEyHaQARWbpZWBgE9404ifrtLLLi97tL1Fwi/YiFsVUqyeognIpHx4mCg1wvTPalPr6LDr/UI9QsGHkM/fAlNpDrsWrTriTiZ4E50EVSfxYDubU567sB6fzO8Df7QNZ6dsr0SPHenDZ7AfwHqmpWobM9I8OBpV2Z0aDdW7PSsoxBO6nsFVA4NcJNuaPU+JMS2GUW2kFt9Dvv/V62qwlFA+z0RLh1bH4Qy0Z+iHR2j9sHnz33Mx0MF+K9a8dyVrVlEVhZkelKmEtUS9Kh8KnYcAH0GJJyw8tMU95 handson-user@cloudshell
```

* `gcp_variables.tf`にあるSSHキーを編集します。

```bash
vi $HOME/handson-wp/gcp_variables.tf
```

* `gcp_variables.tf`内のソースを一部変更します。

ソース内の記載内容を一部変更します。

variables "wp\_ssh\_keys"内の、**username**と**ssh-rsa xxxx**を変更します。  
**username**には、cloud shell上で表示されている@cloud shellより前の値を入力してください。  

変更例は下記の通りです。

```
variable "test_ssh_keys" {
...
username: ssh-rsa xxxxx(id_rsa.pubをコピーした内容)
↓
yoshiki_tada: ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCy+7N4kisvLVs9FcaRUhozkwxB8Ak0AlwVbw7EQTkFmWwEyHaQARWbpZWBgE9404ifrtLLLi97tL1Fwi/YiFsVUqyeognIpHx4mCg1wvTPalPrrueoiwuaj1bH4Qy0Z+iHR2j9sHnz33Mx0MF+K96LDr/UI9QsGHkM/fAlNpDrsWrTriTiZ4E50EVSfxYDubU567ssaB6fzO8Df7QNZ6dsr0SPHenDZ7AfwHqmpWobM9I8OBpV2Z0aDdW7PSsoxBO6nsFVA4NcJNu201j81026/aPU+JMS2GUW2kFt9Dvv/V62qwlFA+z0RLh1bH4Qy0Z+iHR2j9sHnz33Mx0MF+K9a8dyVrVlEVhZkelKmEtUS9Kh8KnYcAH0GJJyw8tMU95 handson-user@cloudshell
...
EOF
}

```

次に、variables "userNumber"内の**default**の値を変更します。  
現在`000000`と設定されていますが、この数字を**ご自身の好きな数字**に変更してください。  

変更例は下記のとおりです。

```

variable "userNumber" {
  default = "0000000"
  ↓
  default = "0123456"
}

```

## Terraformを利用したインスタンス作成

### Terraformを利用してGCE、GLB、SQLを設定する

以下のコマンドを入力し、Terraformを実行します。
約10分程でGCE、GLB、CloudSQLの設定と購入が可能となります。

terraformの初期化(GCPのリソースを利用することを宣言しています)

```bash
terraform init
```

terraformの構文エラー確認(エラーがなければ何も表示されません)

```bash
terraform validate
```

terraformのdry runの実行(実行した際の反映結果の確認)

```bash
terraform plan
```

terraformの実行(実行後`yes`を入力してください)

```bash
terraform apply
```

実行後、以下のサービスが作成されたかの確認をしてください。
* GCEインスタンス
* インスタンスグループ
* GLB
* CloudSQL

## Ansible利用のための準備
所要時間<walkthrough-tutorial-duration duration="10"></walkthrough-tutorial-duration>

### 対象サーバにpython,python-aptをインストールする

Ansibleを利用するための条件として、以下の条件があります。
* 管理対象サーバ(作成したGCEインスタンス)にpython2.4以上がインストールされている
+ 管理サーバ(Cloud Shell)から管理対象サーバに対してSSH接続が可能である

今回は管理対象サーバがUbuntuのため、デフォルトでpythonがインストールされていません。そのため、pythonをインストールします。
対象サーバに対してSSH接続を行います。ユーザ名、IPアドレスについては適宜変更してください。
```
ssh user@11.22.33.44 -i ~/.ssh/id_rsa
```

インスタンスログイン後、下記コマンドを入力してください。

```bash
sudo apt-get install -y python python-apt
```

実行後、ログアウトします。

```bash
exit
```

## Ansible利用のための準備

### 対象ホストファイルを作成
Ansibleには、どのサーバに対し設定を行うかを定義するファイルとしてinventoryファイルがあります。
inventoryファイルの作成を行うため、handson-wp/ansible配下に移動します。

```bash
cd ~/handson-wp/ansible
```

次に、ansibleディレクトリ配下にinventoryファイルを作成します。
IPアドレスは作成したGCEインスタンスの外部IPアドレスを記載してください。
IPアドレスについては適宜変更してください。

```
cat << EOS >> ./inventory
[wordpress]
22.33.44.55
EOS
```

## Ansible利用のための準備

### Ansibleモジュールからping疎通確認

pythonモジュールのインストール後、GCEインスタンスへAnsibleを経由して接続できるかの確認を行います。
以下コマンドを利用して、Ansibleモジュールの実行確認をしてください。

```bash
ansible -i inventory wordpress -m ping
```

実行完了時、以下のような返答が表記されます。

```
SUCCECC => {
    "changed": "false",
    "ping" : "pong"
}
```

## Ansibleを利用したWordpress導入
所要時間<walkthrough-tutorial-duration duration="10"></walkthrough-tutorial-duration>


### Ansibleを実行する
今回演習で利用するファイルとして、2つのPlaybook(手順書)があります。
* common.yml
  * パッケージディレクトリのアップデートやnginxのインストール、初期設定といった共通内容を記載
* wordpress.yml
   * wordpressをインストールし初期設定を実施

以下のコマンドを実行し、それぞれのplaybookを実行します。

```bash
ansible-playbook -i inventory common.yml
```

common.ymlの実行が終わったあと、wordpress.ymlを実行します。

```bash
ansible-playbook -i inventory wordpress.yml
```

## Ansibleを利用したWordpress導入
### Wordpressの動作をブラウザ上にて確認

先程と同様、ロードバランサのページに遷移し、作成したロードバランサのフロントエンド側のURLをコピーし、Wordpressの起動確認を行います。


## Ansibleを利用したWordpress導入
### 演習リソースの削除

演習で利用したリソースについては、Cloud Shell上で下記のコマンドを実行すると削除ができます。実行時確認がありますので、yesと入力してください。
すべてのリソース削除については約5分程度かかります。

```bash
terraform destroy
```



## Conclusion

演習お疲れ様です。Google Cloud Platform では多くのサービスを提供しています。今回作成した演習課題を応用して行う課題を以下に記載しておきますのでぜひ挑戦してください。

### 課題：Wordpress のDatabaseをローカルのMySQLからCloudSQLに変更する（SaaSを使ってみる）
ヒント
* CloudSQLのオーダ
* 既存のMySQLをバックアップ
* CloudSQLにリストア
* Wordpressの接続先変更

### 課題：Wordpresサーバ（GCE）をマルチリージョンで構成する。現在のus-east1-b以外に us-east1-c でもインスタンスを動かしてみましょう
ヒント
* GCEをスナップショット
* GCEをリストア
* インスタンスグループの設定

### 課題：ログを確認しよう。Google Cloud Platform で発生するログは Stackdiverに保管されています
