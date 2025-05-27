---
title: 日本郵政「デジタルアドレス」を使ってみた感想
tags:
  - PHP
  - RestAPI
  - デジタルアドレス
  - 日本郵政
private: true
updated_at: '2025-05-27T13:13:57+09:00'
id: dfce603f474a941ae3e1
organization_url_name: null
slide: false
ignorePublish: false
---

# はじめに

2025年5月26日、日本郵政が新しいサービス「[デジタルアドレス](https://lp.da.pf.japanpost.jp/)」を開始しました。

エンジニア泣かせな日本の住所表記を英数字7桁で表現することができ、表記ブレを無くすことができる画期的な取り組みです。

同時に「[郵便番号・デジタルアドレスAPI](https://lp-api.da.pf.japanpost.jp/)」もリリースされました。

このAPIを使ってみた感想と利用方法について紹介します。

# デジタルアドレスとは

デジタルアドレスは、日本郵政が提供する新しい住所表現システムです。例えば「東京都千代田区丸の内2丁目7-2部屋番号:サンプル1」という長い住所を、「A7E2FK2」という7桁の英数字で表現できます。

## 主な特徴

- **7桁英数字**：覚えやすく、入力しやすい
- **無料取得**：ゆうIDとの連携で無料で取得可能
- **継続利用**：引越し後も同じアドレスを継続使用可能
- **詳細住所対応**：部屋番号まで含む詳細な住所情報

# 実際に使ってみた感想

## 取得プロセス

デジタルアドレスの取得は思っていたより簡単でした。ゆうIDを持っていれば、専用サイトから申請するだけです。申請後、数分で7桁のデジタルアドレスが発行されました。

## 利便性

最も感じたのは、住所入力の時短効果です。特に郵便局アプリでの送り状作成では、デジタルアドレスを入力するだけで、部屋番号まで含む完全な住所が自動入力されます。これまで何度も入力していた長い住所を、7文字で済ませられるのは革新的です。

## 現時点での制限

ただし、現時点では制限もあります：

- **認知度の低さ**：サービス開始直後で、まだ広く知られていない
- **対応サービスの少なさ**：まだ対応しているサービスが限られている
- **郵便物送付の制限**：デジタルアドレスのみでは郵便物を送れず、そこから照合した郵便番号、住所、氏名が必要です。

# 開発者向け：API利用の詳細

デジタルアドレスは、API経由でシステムに組み込むことも可能です。ただし、いくつかの重要な注意点があります。

## 利用条件

**重要：API利用は法人・個人事業主限定です。**

- 「郵便番号・デジタルアドレス for Biz」への登録が必要
- 法人番号などの事業者情報が必要

## 事前準備

API利用前に以下の設定が必要です：

1. **法人登録**：事業者としての登録
   - 管理画面から「法人登録」を選択
   - 必要事項を入力（法人番号、会社名、代表者名など）
   - 登録完了後、APIキーが発行されます

![法人登録画面](https://i.gyazo.com/9cc578502e45ba095c283202e69a4b8b.png)


2. **システムリスト登録**：管理画面で以下を設定
   - システム名
   - URL（アクセス元ドメイン）
   - IPアドレス（最大10件）

![システムリスト登録画面](https://i.gyazo.com/c4ae54b56d0635b08a56fff2974439b5.png)


**注意**：システムリストの設定は管理者権限のみ可能で、一般ユーザは設定できません。

## 技術的制限

### CORS制限
フロントエンド（JavaScript）から直接APIを呼び出すことはできません。

```javascript
// ❌ ブラウザから直接呼び出すとCORSエラーが発生
fetch('token取得用APIエンドポイントURL', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    grant_type: 'client_credentials',
    client_id: 'クライアントID',
    secret_key: 'クライアントシークレット'
  })
})
.then(response => response.json())
.then(data => console.log(data))
.catch(error => console.error('CORS Error:', error));
```

**対処法**：
- バックエンド（PHP、Node.js等）でプロキシAPIを作成
- サーバーサイドでAPIを呼び出し、結果をフロントエンドに返す

フロントエンドでも呼び出せるようにCORS対応版クライアントAPIを開発された方がいらっしゃいます。<br />
[CORS対応版 郵便番号・デジタルアドレスAPIを開発しました digital-address.app ゆうID不要](https://qiita.com/relu/items/9b8085e89c01bcf6d35a)

## API利用手順

API利用は2段階のプロセスです：

### 1. トークン取得

```php
<?php
$tokenUrl = 'token取得用APIエンドポイントURL';
$requestData = [
  'grant_type' => 'client_credentials',
  'client_id' => 'クライアントID',
  'secret_key' => 'クライアントシークレット'
];

$curlHandle = curl_init($tokenUrl);
curl_setopt_array($curlHandle, [
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_POST => true,
  CURLOPT_POSTFIELDS => json_encode($requestData),
  CURLOPT_HTTPHEADER => [
    'Content-Type: application/json',
    'User-Agent: Mozilla/5.0 (compatible; PHP cURL)',
    'X-Forwarded-For: ' . $_SERVER['REMOTE_ADDR']
  ]
]);

$tokenResponse = curl_exec($curlHandle);
$tokenResult = json_decode($tokenResponse, true);
curl_close($curlHandle);
?>
```

:::note alert
User-Agentヘッダーは公式ドキュメントに必須と明記されていませんが、指定しないと403エラーが発生しました。
:::

### 2. 住所検索

```php
// トークン取得後
if (isset($tokenResult['token'])) {
  $digitalAddress = '7桁のデジタルアドレス';
  $accessToken = $tokenResult['token'];
  $searchUrl = '住所取得用APIエンドポイントURL' . $digitalAddress;

  $searchHandle = curl_init($searchUrl);
  curl_setopt_array($searchHandle, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER => [
      'Authorization: Bearer ' . $accessToken,
      'User-Agent: Mozilla/5.0 (compatible; PHP cURL)'
    ]
  ]);

  $searchResponse = curl_exec($searchHandle);
  $searchResult = json_decode($searchResponse, true);
  curl_close($searchHandle);
}
?>
```

## レスポンス例

### トークン取得レスポンス
```json
{
  "token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "jwt",
  "expires_in": 600,
  "scope": "J1"
}
```

- トークンの有効期限は600秒（10分）
- JWT形式で619文字程度の長い文字列

### 住所検索レスポンス
```json
{
  "addresses": [{
    "dgacode": "A7E2FK2",
    "zip_code": "1000005",
    "pref_name": "東京都",
    "city_name": "千代田区",
    "town_name": "丸の内",
    "block_name": "二丁目７－２",
    "other_name": "サンプル１",
    "address": "東京都千代田区丸の内二丁目７－２サンプル１"
  }]
}
```

部屋番号まで含む詳細な住所情報が取得できます。

# 実運用での課題

## 権限管理の問題：組織は誰が作る？

APIの利用は無料なのでこちらで代理で組織を作っても良いが、事業者番頭等の入力も必要なことや、ゆうIDとの連携が必要になるため、クライアントに作ってもらう方がベター。<br />
↓<br />
組織へのユーザ招待機能はあるものの、システムリストの追加やユーザ招待は管理者権限が必要。<br />
↓<br />
そうするとクライアントにシステムリストを登録してもらうことになる。<br />
↓<br />
IPアドレスの登録もあるので嫌われがち。<br />
↓<br />
↓<br />
↓<br />
現場レベルでは開発者権限でシステムリストの増減くらいはさせてほしい。

## セキュリティ面の懸念

- **総当たり攻撃**：7桁という短さから、総当たり攻撃の可能性
- **匿名性の課題**：デジタルアドレスから住所が検索可能なため、匿名配送には不向き
- **対策の限界**：現在の対策は利用規約レベルに留まっている

# 今後の展望

## 普及への期待

- **大手企業の導入**：[楽天やGMOなどが導入を検討中](https://www.nikkei.com/article/DGKKZO88914900W5A520C2MM8000/)
- **多様な活用場面**：[タクシーやカーナビでの利用可能性](https://www.asahi.com/articles/AST5V1TFZT5VULFA00RM.html?iref=comtop_BreakingNews_list)
- **社会インフラ化**：[10年構想での社会基盤としての位置づけ](https://www.watch.impress.co.jp/docs/news/2017226.html)

## 改善への期待

- **個人開発者向けプラン**：現在は法人限定だが、個人向けプランの提供
- **権限管理の改善**：開発者権限の実装
- **対応サービスの拡大**：より多くのWebサービスでの対応

# 他サービスとの比較

海外には類似サービス（What3Wordsなど）がありますが、日本郵政のデジタルアドレスは以下の特徴があります：

- **公的機関による運営**：信頼性の高さ
- **既存インフラとの連携**：郵便システムとの統合
- **日本独自の住所体系**：日本の住所表記に最適化

# まとめ

デジタルアドレスは、住所入力の手間を大幅に削減する画期的なサービスです。現時点では制限や課題もありますが、将来的な可能性は非常に大きいと感じました。

特に開発者にとっては、API経由での住所検索機能は魅力的です。ただし、法人限定という制限や権限管理の課題があるため、これらの改善が普及の鍵となるでしょう。

サービス開始直後ということもあり、今後の発展と改善に期待したいと思います。住所という日常的な情報がデジタル化されることで、私たちの生活がより便利になる日が近づいているのかもしれません。

---

# 参考資料

- [日本郵便公式プレスリリース](https://www.post.japanpost.jp/notification/pressrelease/2025/00_honsha/0526_01.html)
- [デジタルアドレス公式サイト](https://lp.da.pf.japanpost.jp/)
- [郵便番号・デジタルアドレスAPI](https://guide-biz.da.pf.japanpost.jp/api/)

*この記事は2025年5月27日時点の情報に基づいています。サービス内容は今後変更される可能性があります。* 
