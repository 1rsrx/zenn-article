---
title: "ConnectへのアップロードでThis bundle does not support one or more of the...が発生"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["swift"]
published: true
---

## 発生したこと
他の会社が開発していたアプリを上書きアップデートする案件で、
AppStoreConnectにアップしようとしたところ下記エラーが発生しました。

```
Asset validation failed

This bundle does not support one or more of the devices supported by the previous app version.
Your app update must continue to support all devices previously supported.
You declare supported devices in Xcode with the Targeted Device Family build setting. 
Refer to QA1623 for additional information: https://developer.apple.com/library/ios/#qa/qa1623/_index.html (ID: 47123885-ca9b-415f-b240-994f420af5bd)
```

## 原因
元アプリでは、iPhone, iPadが対応されていました。
新規に開発したアプリは、iPhoneしか想定しておらず、iPadは対応デバイスに登録されていませんでした。
既にiPad対応されており、アップデートでiPadを対応端末から削除することはできないため、エラーが発生したようです。

## ソース
 https://developer.apple.com/library/ios/#qa/qa1623/_index.html
 
 エラー内のリンクには、下記が書かれていました。
```
(質問)
Why am I getting an error about supporting previous devices when I upload an update of my app?

アプリのアップデートをアップロードすると、以前のデバイスのサポートに関するエラーが表示されるのはなぜですか？

(回答)
iTunes Connect does not allow uploading an updated version of an app when the update runs on fewer devices than the version of the app currently in the App Store. This is by design.

An update to an app must work for every customer who has already purchased the app, and is running a current version of iOS.
iTunes Connectは、現在App Storeにあるアプリケーションのバージョンよりも少ないデバイスで実行されるアップデート版アプリケーションのアップロードを許可しません。これは意図的なものです。
アプリケーションのアップデートは、すでにそのアプリケーションを購入し、現在のバージョンのiOSを実行しているすべてのお客様に対して動作する必要があります。
 ```

対応端末を減らすことは出来ないようですね。

 ## 対応策
 - iPad対応する
 - ストアからアプリを削除し、新アプリとしてリリースする