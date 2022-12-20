---
title: "Realmを使ったユニットテストを書いてみた"
emoji: "🦁"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["swift", "ios"]
published: true
---

# 対象
- テストコードがなんたるかは、理解したけど、具体的なユースケースが分からない人

# 準備
テストしたい関数を定義していきます。

## 関数の定義
```swift
class Record: Object {
    dynamic var id = ""
    dynamic var score = 0

    convenience init(id: String, score: Int) {
        self.init()

        self.id = id
        self.score = score
    }
}

func getRrecords(between start: Int, and end: Int) -> [Record] {
  let records = realm.objects(Record.self)
            .filter("%@ <= score AND score <= %@", start, end)
            .sorted(byKeyPath: "score")
        
  return Array(records)
}
```

scoreが指定した範囲内にあるデータを取得する関数を定義しました。


## 全体
全体のコードです。
ViewController内のrealmはinjectメソッドで設定するようにしています。
(ここでテスト用のRealmインスタンスとアプリを動かすときのRealmインスタンスを分けるため)

```swift
import UIKit
import RealmSwift

@objcMembers
class Record: Object {
    dynamic var id = ""
    dynamic var score = 0

    convenience init(id: String, score: Int) {
        self.init()

        self.id = id
        self.score = score
    }
}

class ViewController: UIViewController {

    private var realm: Realm!

    override func viewDidLoad() {
        super.viewDidLoad()

        let todos = getRrecords(between: 80, and: 100)
        print(todos)
    }

    func inject(realm: Realm) {
        self.realm = realm
    }

    func getRrecords(between start: Int, and end: Int) -> [Record] {
        let records = realm.objects(Record.self)
            .filter("%@ <= score AND score <= %@", start, end)
        
        return Array(records)
    }
}

class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let scene = (scene as? UIWindowScene) else { return }
        let window = UIWindow(windowScene: scene)

        let rootViewController = ViewController()
        let realm = try! Realm()

        // realmオブジェクトの注入
        rootViewController.inject(realm: realm)

        window.rootViewController = rootViewController
        window.makeKeyAndVisible()

        self.window = window
    }

    // (略)
}
```

# テストコード

## テストの要件
- 指定した範囲のデータが取得できること
  - 90 ~ 100でテスト

## 書いていく
```swift
import XCTest
import RealmSwift
@testable import {アプリ名}

final class ViewControllerTests: XCTestCase {
    private let viewController = ViewController()
    private var realm: Realm!

    override func setUpWithError() throws {
        // https://www.mongodb.com/docs/realm-legacy/jp/docs/swift/latest.html#%E3%83%86%E3%82%B9%E3%83%88
        // Realmの公式ドキュメントに従い、設定を変更する
        Realm.Configuration.defaultConfiguration.inMemoryIdentifier = self.name

        realm = try! Realm()
        viewController.inject(realm: realm)
    }

    override func tearDownWithError() throws {
        reset()
    }

    // この中にテストコードを書く
    func testGetRrecords() throws {

    }

    // テストデータ追加用の関数
    private func addRecords() {
        for i in 0...100 {
            let record = Record(id: NSUUID().uuidString, score: i)
            try! realm.write {
                realm.add(record)
            }
        }
    }
}
```

## テスト用の関数
```swift
func testGetRrecords() throws {
    addRecords()

    // 範囲の最小値と最大値を決める
    let startScore = 90
    let endScore = 100

    let records = viewController.getRrecords(between: startScore, and: endScore)
    let max = records.map { $0.score }.max()!
    let min = records.map { $0.score }.min()!

    // 取得したレコード一覧の最大値最小値が想定通りか検証
    XCTAssertEqual(100, max)
    XCTAssertEqual(90, min)
}
```

## 全体
```swift
import XCTest
import RealmSwift
@testable import {アプリ名}

final class RealmSampleTests: XCTestCase {

    private let viewController = ViewController()
    private var realm: Realm!

    override func setUpWithError() throws {
        Realm.Configuration.defaultConfiguration.inMemoryIdentifier = self.name

        realm = try! Realm()
        viewController.inject(realm: realm)
    }

    override func tearDownWithError() throws {
        reset()
    }

    func testExample() throws {
        addRecords()

        let startScore = 90
        let endScore = 100
        let records = viewController.getRrecords(between: startScore, and: endScore)

        let max = records.map { $0.score }.max()!
        let min = records.map { $0.score }.min()!

        XCTAssertEqual(100, max)
        XCTAssertEqual(90, min)
    }

    private func addRecords() {
        for i in 0...100 {
            let record = Record(id: NSUUID().uuidString, score: i)
            try! realm.write {
                realm.add(record)
            }
        }
    }

    private func reset() {
        try! realm.write {
            realm.deleteAll()
        }
    }
}
```

# 説明を書略したこと
- DI
  - https://qiita.com/eKushida/items/78a58559aedd851d105c
- 既存プロジェクトにユニットテストの追加
  - https://dev.classmethod.jp/articles/yunito-tesuto/

# 感想
業務でテストコードはあまり書かないのですが、
個人アプリを叩き台にして、練習してみました。

テストコードがあると、安心できますし、バグの切り分けをしやすいくなるかなと思います。
(業務のでも導入したい...)

最後まで読んでいただき、ありがとうございました！