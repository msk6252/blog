---
title: cloud watch eventsでcronを設定するときはタイムゾーンに注意を！
url: 116.html
id: 116
categories:
  - AWS
  - CloudWatch
  - lambda
  - インフラ
date: 2019-11-10 23:28:14
tags:
---

特に忙しいわけでもなく、ただブログを放置していました。 serverless framework + lambda + line apiを使って、電車の遅延通知を作成していました。 そこで得た知見を共有したいと思います。

事象
--

lambda + cloud watch eventsで定期的にcronで関数を起動するような仕組みを作成しました。 軽いテストも行い、Lineに電車の遅延の通知が来ることも確認しました。 しかし、電車の遅延があった日にLineを確認しても、届いていないのです。 叩いているAPIに問題があるのかとも思いましたが再度テストをし、動作を確認するも正常に動作がする。 これは、lambda側に問題はないということが分かりました。

原因
--

タイトルどおりですが、cloud watch eventsのcron式はutc時間で記述するべきだったようです。 8時に実行するようにcron式を書いていましたが、日本時間では17時に実行されていました。 UTC時間で記述する場合は実行してほしい時間の-9時間なので23時に設定するべきだったみたいです。 serverless.yml内を以下のように書き換え

    schedule: cron(0 8 * * ? *)
    ↓
    # (24 + 8) - 9時間 = 23時
    schedule: cron(0 23 * * ? *)