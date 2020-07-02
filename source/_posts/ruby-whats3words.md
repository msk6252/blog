---
layout: blog
title: Rubyでwhat3words APIを使ってみる
date: 2020-07-02 12:53:31
tags:
---

# What3WordsのAPIを使って自分の住所の3wordを取得する
## はじめに
Rubyは気軽に書けて実行もすぐできるでいいですよね。
ということで、今回は外部のAPIを使って簡単なスクリプトを作ってみたいと思います。
今回使うAPIはWhat3wordsという位置情報サービスのAPIを使います。
今回作るものはエラー処理とか考慮していないのでちゃんと作りたい場合は、別途エラー処理を加えてください

## What3wordsとは
What3wordsとは、世界中の住所を3つの単語で組み合わせて表現するものです。
世界中を3m ✕  3mの正方形に区切るため、従来の住所より細かい位置を指定することができます。
余談ですが、京都の住所は長いと言われてますが碁盤の目の形を活かして、通りの名前 +「上ル」や「下ル」という組み合わせになっているそうです。そのため、この京都の違うところに住んでいる人に住所を伝えても大体の位置がわかるという話を聞いたことがあります。
What3wordsを使えば、京都の長い住所も3語で表現することができます。
まぁ、人にこの3語を伝えても位置は分からないわけですが。。。

## 今回作るスクリプト
今回作るスクリプトとは、住所から3単語を出すというものです。
下記のような感じです。

住所入力 → (API) www.geocoding.jp → (API) what3words → ３単語

## スクリプト
```
#!/usr/bin/env ruby
require 'net/http'
require 'uri'
require 'json'
require 'rexml/document'

if ARGV[0].nil?
  exit 1
end

address = ARGV[0]

def get_geocoding(address)
  # 住所を引数にして、問い合わせ
  url = URI.encode("https://www.geocoding.jp/api/?q=#{address}")
  uri = URI.parse(url)
  xml = Net::HTTP.get(uri)

  # 返ってきたXMLをパースし、緯度経度を抜き出す
  doc = REXML::Document.new(xml)
  return doc.elements['result/coordinate/lat'].text, doc.elements['result/coordinate/lng'].text
end

# 抜き出した緯度経度を引数にし、what3wordsに問い合わせを行う
# [APIKEY]の部分はwhat3wordsで取得する必要がある
def get_what3words(lat, lon)
  uri = URI.parse("https://api.what3words.com/v3/convert-to-3wa?coordinates=#{lat}%2C#{lon}&language=ja&key=J14C8112")
  json = Net::HTTP.get(uri)
  result = JSON.parse(json)
end

# 各関数を呼び出す
lat, lon = get_geocoding(address)
json = get_what3words(lat, lon)

# 返ってきたjsonから単語だけを抽出する
puts json["words"]
```

実行/実行結果
```
ruby test.rb 東京都港区芝公園４丁目２−８
=> けんぶつ・このは・はかり
```

## まとめ
こんな感じで住所から3単語を抽出できました。
ちなみに実行時に指定した住所は東京タワーのものです。
観光地だけに「けんぶつ」が含まれてますね！意図的なものだろうか気になる...
