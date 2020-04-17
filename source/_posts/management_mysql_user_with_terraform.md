---
title: Terraformを使ってMySQLのDBユーザを管理する
---
## はじめに
コロナウイルスの影響もあり最近は家にいる時間が明らかに増えてきました。
今までのようには出かけられなくなって残念という気持ちの反面、自分への投資ということで最近は投資や技術の勉強に勤しんでいます。
DBのユーザの管理方法を見直すということで、Terraformを使った管理方法を試していました。
あまり記事が出ていなかったのでこちらに残します。

## 作業準備
作業を行うディレクトリを作成します。
```
# 作業用ディレクトリを作成、移動
mkdir ~/mysql_management && cd $_
```

## tfenvを使ったTerraformのインストール
brewでの管理はあまり好きではないので今回はgithub上のリポジトリからクローンしてきます。
```
git clone https://github.com/tfutils/tfenv.git ~/.tfenv
cd ~/.tfenv
# 最新でエラーが出たのでバージョンを落として使用
git checkout v1.0.2
```

パスを通す
```
# bashの場合
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.bash_profile

# zshの場合
echo 'export PATH="$HOME/.tfenv/bin:$PATH"' >> ~/.zsh_profile
```

各ディストリビューションごとの設定については、細かい設定については[tfenvのREADME](https://github.com/tfutils/tfenv)を参照ください。

これでtfenvが使えるようになると思います。

次はTerraform自体のインストール
```
tfenv install 0.11.14
tfenv use 0.11.14
terraform --version # バージョンが表示されたらOK!
```

## DockerでMysql環境を作成する
今回は、お試しということでMySQL環境をDockerで立ち上げます。

```
docker pull MySQL
docker run -it --name test_db -e MYSQL_ROOT_PASSWORD=root -d mysql:latest

# 立ち上げたコンテナ上にログイン
docker exec -it test_db bash -p

# コンテナ上でmysqlをユーザを確認
mysql -u root -proot
use mysql;
select * from user\G;
```

## PGP鍵を使ったパスワード生成
まず、GPGのインストール
```
brew install gpg
```

次は公開鍵、秘密鍵の作成
```
gpg --gen-key

# 本名など聞かれますが、今回はtest_userで作成
# そのほかメールアドレスとは空欄でスキップ
本名: test_user
...
名前(N)、電子メール(E)の変更、またはOK(O)か終了(Q)?: O

# 画面が変わり鍵にかけるパスフレーズの設定を促す画面が表示される
# 任意のパスフレーズを入力（２回入力する）
# うまくいけば鍵が生成される
```

鍵の出力
```
# 公開鍵をファイルとして出力
gpg --output test_user.public.gpg --export test_user

# 秘密鍵をファイルとして出力
gpg --output test_user.private.gpg --export-secret-keys test_user

# 出力されたファイルを確認
# 公開鍵と秘密鍵の２つができていればOK
ls *.gpg

# Terraformに設定するため公開鍵をbase64にエンコードしたものを取得
# 大量の文字列が表示されるのでコピー
cat test_user.public.gpg | base64

```

PGP鍵についてこれにて完了

## Terraformの設定
長い道のりでしたが、ようやく本題です。
Terraformのファイルを作成していきます。

```
variable "pgp_key" {
  # 先程の公開鍵をエンコードしたもの(cat test_user.public.gpg | base64のやつ)
  default = "" 
}

provider "mysql" {
  # DBのエンドポイント
  endpoint = "127.0.0.1:3306"
  # ユーザを書き込むユーザ（今回はroot）
  username = "root"
　# ユーザを書き込むユーザのパスワード
  password = "root"
}

# 追加するユーザを記述
resource "mysql_user" "chunli" {
  # ユーザ名
  user = "chunli"
  # Userテーブルのhostに設定される値
  host = "%"
  # variableに設定したpgp_keyが設定される
  pgp_key = "${var.pgp_key}"
}

output "encrypted_password" {
  value = "${mysql_user_password.chunli.encrypted_password}"  
}
```

## Terraformを実行してみる
ここまでで準備ができました。
Terraformを実行してみましょう。
```
# 初回はinitを行う
terraform init

# dry-run(DBには反映されず、結果のみ表示)
terraform plan

# run(実行)
terraform apply

```

結果を確認してみましょう
```
# パスワードを取得する
terraform output encrypted_password | base64 -D | gpg -d

# 立ち上げたコンテナ上にログイン
docker exec -it test_db bash -p

# コンテナ上、先程作ったユーザと取得したパスワードでログイン
# ログインできたらOK
mysql -u chunli -p
```

## まとめ
TerraformでMySQLユーザの管理を行ってみました。
PGPの部分はkeybaseを使ってパスワードを生成することもできるみたいなので、
気になるかたはやってみてください。
