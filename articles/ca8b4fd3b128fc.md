---
title: "iPhone14Proでカメラのピントが合わない時の対処法"
emoji: "🐾"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["swift"]
published: true
---

# 原因
- iPhone14Pro, 14ProMaxはピントが合う距離が他の端末より長い

# 問題点
- 距離を離すことでピント自体は合うが、撮影対象が小さく映る
- さらに私の場合、開発中のアプリにはバーコードスキャン機能があり、ピントが合う距離まで離すと、対象が小さすぎて認識できない

# 結論
- ズームさせる

# ZXingObjCの例
```swift
extension UIDevice {
    var modelName: String {
        var systemInfo = utsname()
        uname(&systemInfo)
        let machine = withUnsafePointer(to: &systemInfo.machine) {
            $0.withMemoryRebound(to: CChar.self, capacity: 1) {
                ptr in String.init(validatingUTF8: ptr)
            }
        }
        return machine ?? "Unknown"
    }
}


let zxcapture = ZXCapture()

let deviceName = UIDevice.current.modelName

// iPhone14Pro, 14ProMaxのみズームさせる
// https://newichiros-memo.blogspot.com/2019/12/iphone-devicename.html
if deviceName == "iPhone15,2" || deviceName == "iPhone15,3" {
    do {
        try zxcapture.captureDevice.lockForConfiguration()
        zxcapture.captureDevice.videoZoomFactor = 2.5 // 2.5倍にズーム
    } catch {}
}
```

# 純正カメラの挙動
14Proの純正カメラは近距離まで近づくと、自動でマクロ撮影モードに切り替えることができるため、この問題は起こらない
