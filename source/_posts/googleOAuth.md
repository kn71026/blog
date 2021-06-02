---
title: GoogleOAuth
date: 2021-06-02 14:35:49
tags: 
- web
- Google OAuth
categories: api
toc: true
---


剛好最近在寫專題的登入功能，有接到google的OAuth，紀錄一下中間遇到的問題，跟最後跑起來的流程。
api測試工具：Postman

## 參考文件
- [ Google官方文件](https://developers.google.com/identity/protocols/oauth2)
- [Google官方前端範例code](https://developers.google.com/identity/sign-in/web)

## 運作流程

因為前端的部分還沒完成，所以先以能用後端測試的方式去實作
![運作流程](https://raw.githubusercontent.com/kn71026/image/master/image/%E6%B5%81%E7%A8%8B.jpg)
<!-- more -->
1. 前端點擊登入按鈕後，顯示大家常看到的google登入頁面
2. 在登入成功後，google跳轉到特定頁面(後面會設定)，並回傳token
3. 前端call後端寫的api，並將token一起傳過去
4. 後端接收到token後，再call一次google的oauth api，並傳入前端給的token做二次的驗證
5. 如果token(沒有遭到串改或過期)，根據token可拿到的scope回傳特定範圍的資料
6. 後端擷取有用的資料後，存到自己專案的DB，並回傳資料給前端

在過程中，因為前端跟後端之間基本上是拿token做溝通的，所以也不需要將使用者的密碼存到DB中

## Prerequirement

- 先到[GCP](https://console.cloud.google.com/home/)建立一個新專案，之後要在裡面做設定
- 建立完成之後，選擇側邊欄的API和服務，並點擊憑證 ![API和服務](https://raw.githubusercontent.com/kn71026/image/master/image/%E5%81%B4%E6%AC%84.png)
- 進到憑證頁面後，按下上方的建立憑證，選擇OAuth用戶端ID
- 先來設定同意畫面，如果是只給機構內的使用者使用(如之前的Gsuite)，可以設定內部，如果是要大家都可以登入，則設定外部
![設定User Type](https://raw.githubusercontent.com/kn71026/image/master/image/%E5%90%8C%E6%84%8F.jpg)
- 並按下建立，開始設定應用程式資訊，以及支援信箱，在頁面的右邊，也顯示了提供的資訊會被如何展示
 ![各種資訊](https://raw.githubusercontent.com/kn71026/image/master/image/%E8%B3%87%E8%A8%8A.jpg)
 ![同意畫面顯示](https://raw.githubusercontent.com/kn71026/image/master/image/%E9%A1%AF%E7%A4%BA%E7%95%AB%E9%9D%A2.jpg)
- 設定授權網域，通常在測試的時候會設定為本地端(需要設定實際測試時才能使用)
- 設定redirect_uri，也就是重新導向的頁面，回傳的token的資訊會在這這段網址之中
- 按下儲存後，設定範圍(scope)，設定你的應用程式能存取那些資訊，這邊設定的是使用者的openid、email、跟帳號資訊(姓、名、頭貼、語系)
  ![設定範圍](https://raw.githubusercontent.com/kn71026/image/master/image/%E7%AF%84%E5%9C%8D.png)
- 在正式發布前，新增測試使用者，在發布前，要是測試使用者才能登入
  ![設定使用者](https://raw.githubusercontent.com/kn71026/image/master/image/%E6%B8%AC%E8%A9%A6%E4%BD%BF%E7%94%A8%E8%80%85.png)
- 最後回到憑證頁面，下載設定檔
![下載設定檔](https://raw.githubusercontent.com/kn71026/image/master/image/%E8%A8%AD%E5%AE%9A%E6%AA%94.jpg)

到這邊為止，前置作業就完成啦，在設定檔中，主要會用到的有：``client_id``、``client_secret``

## 運作流程

在開始使用之前，[官方的文件](https://developers.google.com/identity/protocols/oauth2/javascript-implicit-flow#oauth-2.0-endpoints)中提供了各種參數的詳細介紹。

以這次要用到的OAuth 2.0為範例，有些參數會設置為必填：

| Parameters |  |  |
| -------- | -------- | -------- |
| `client_id`  | Required | 專案設定檔中的client_id |
| `redirect_uri`  | Required   | user 成功登入後 redirects 的  URI  |
| `response_type`   | Required  |  回傳格式，這邊設為token |
| `scope`    | Required  | 允許存取的範圍    |


### 執行步驟
1. 進到google登入的畫面(可手刻按鈕或是使用官方提供的素材)
2. 可以看一下在登入畫面的網址中有哪些資訊
![URL內容](https://raw.githubusercontent.com/kn71026/image/master/image/url.png)
3. 若成功登入後，則會導向前面設定的`redirect_uri`，並拿到回傳的token
![回傳的token](https://raw.githubusercontent.com/kn71026/image/master/image/token.png)
4. call OAuth api 拿到使用者的資料
![呼叫成功!](https://raw.githubusercontent.com/kn71026/image/master/image/postman.png)

## 後續處理
之後就能在自己後端的code中，以類似的方式去串接Google帳號了🎉🎉🎉