# 來找我

傳一個連結給家人，他們跟著大箭頭走，就會走到你身邊。專門給不會看地圖的長輩用。

網址：https://bruce-hsieh.github.io/find-me/

## 怎麼用
1. 分享的人：打開 App → 填稱呼（例如「兒子」）、手機號碼（選填）→「開始分享位置」→「用 LINE 傳連結」
2. 收到的人：點連結（會自動改用 Safari 開）→ 按「開始找」→ 兩次都按「允許」→ 跟著箭頭走
3. 碰到面 → 按「結束」，位置資料立刻刪除

## 原理
- 單一 `index.html`，GitHub Pages 上線
- 位置透過 Firebase Realtime Database（專案 `find-me-2ee50`，新加坡機房）即時交換：`rooms/<房間碼>/a`（分享的人）、`rooms/<房間碼>/b`（打開連結的人），每 2 秒最多寫一次
- 房間碼是 24 碼亂數，1 小時後失效
- 箭頭方向 = 對方方位角 − 手機羅盤朝向（iPhone 用 `webkitCompassHeading`）

## Firebase 安全規則
```json
{
  "rules": {
    "rooms": {
      "$room": {
        ".read": "$room.length >= 20",
        "$who": {
          ".write": "$room.length >= 20",
          ".validate": "!newData.exists() || (newData.child('lat').isNumber() && newData.child('lng').isNumber() && newData.child('t').isNumber())"
        }
      }
    }
  }
}
```

## 改版流程
1. 改 `index.html`
2. 進版時 `APP_VERSION`、`CHANGELOG`、`version.json` 三個版號一起改
3. 換圖示或啟動圖要換檔名（`v1-` → `v2-`），並重新加入主畫面

## 踩過的坑
- iPhone 的羅盤、語音、音效都要使用者「親手點一下」才能開，所以全部綁在「開始找」按鈕上
- 從 LINE 點連結會用 LINE 內建瀏覽器，權限常出問題 → 連結加 `openExternalBrowser=1` 讓 LINE 改用 Safari 開

Copyright © 2026 大樹

## 走路線模式
- 路線來源：FOSSGIS 的 OSRM 步行路線（`routing.openstreetmap.de/routed-foot`，OpenStreetMap 資料，免費、不用金鑰）
- 箭頭指向「路線上往前 20 公尺的點」，距離顯示實際步行距離；40 公尺內自動改回直線
- 失敗時自動改用直線並跳出大視窗提醒：逾時 6 秒、起終點離步道超過 60 公尺、直線 <400m 但路線 >4 倍；30 秒後再重試
- 偏離路線 30 公尺或對方移動 30 公尺 → 重新規劃（最快 15 秒一次）
