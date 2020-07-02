---
layout: blog
title: 【Ruby】CSVファイルに読み込み/書き込みを行う
date: 2020-07-01 20:03:59
tags:
---

# RubyでCSVファイルを扱う
## はじめに
CSVファイルを読み込んでデータを整形したり、作ったデータをCSVファイルとして書き込んだりすることもあると思います。
Rubyで書いていきたいと思います。

## 読み込み
下記のようなCSVファイルを準備します。
```
aaa,bbb,ccc
ddd,eee,fff
ggg,hhh,iii
jjj,kkk,lll
```

### CSVの内容すべてを読み込む
```
require 'csv'
CSV.read("test.csv")
=> [["aaa", "bbb", "ccc"], ["ddd", "eee", "fff"], ["ggg", "hhh", "iii"], ["jjj", "kkk", "lll"]]
```

### CSVの内容を１行ずつ読み込む
```
CSV.foreach("test.csv") {|row|
  p row
}
```

## 書き込み
### CSVを作成する
```
CSV.open('hoge.csv', 'w') do |test|
  test << [1,2,3]
  test << [4,5,6]
  test << [7,8,9]
end
```

## モードについて
ファイルの書き込み/読み込みのモードが指定できます。
- "r" 読み込み
- "w" 書き込み
- "b" バイナリモード
- "a" 追記

## まとめ
RubyでCSVのファイルの読み込み/作成をやってみました。
ファイルが存在したら追加、なければ新規作成とかってよしなにやってくれないだろうか...
