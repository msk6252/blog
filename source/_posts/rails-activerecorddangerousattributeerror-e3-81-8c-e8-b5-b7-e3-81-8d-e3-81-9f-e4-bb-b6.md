---
title: '[Rails] ActiveRecord::DangerousAttributeErrorが起きた件'
url: 92.html
id: 92
categories:
  - Rails
  - Ruby
date: 2019-04-07 14:41:05
tags:
---

厄介なエラーが発生してしまった。。。
==================

概要
--

Railsのアップデートを行う機会があり、Rails4.0から4.2にバージョンアップを行っていました。 そこで、ActiveRecord::DangerousAttributeErrorという厄介なエラーが発生してしまいました。

ActiveRecord::DangerousAttributeErrorとはなにか
------------------------------------------

ActiveRecord::DangerousAttributeErrorとは、「ActiveRecordですでに定義されているメソッド名とDB内のカラム名が重複しているため使えないよ」というエラーです。 今回は上記記述でもある通り、Rails4.0から4.2に上げたわけですが、同様の事例が起きたという人がgithubのrailsリポジトリのissuesに立てた人がいました。 https://github.com/rails/rails/issues/18338 issuesによるとRails4.1から4.2に上げた場合に起きたといっております。

解決策は。。。
-------

気になる解決策についてですが、新規プロジェクトについてはカラム名の変更を行うこと。 既存のプロジェクトの場合は、gem「safe_attributes」を利用することで内部的に解消してくれるみたいです。 しかし、最終更新が2013年で止まっています。 また、サポートされているRubyのバージョンが1.9.3です。 今後のRubyのアップデートで動かなくなる可能性もあります。

他の解決策は。
-------

githubのissuesによると下記のメソッドをコメントアウトすることでエラーを止めることができると言っておりますが、 他のところがちゃんと動作するかどうかは保証しないよって言っております。 https://github.com/rails/rails/blob/6d72485b6977804a872dcfa5d7ade2cec74c768e/activerecord/lib/active\_record/attribute\_methods.rb#L131-L133

まとめ
---

結局レガシーなDBに関しても面倒ではありますが、カラム名の変更が一番安全な方法だと考えられます。 今回はmodel_nameというカラムがバッティングしていて、影響範囲が大きいので修正がめんどいです。