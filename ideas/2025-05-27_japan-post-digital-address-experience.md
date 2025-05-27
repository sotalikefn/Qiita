# 日本郵政「デジタルアドレス」を使ってみた感想 - 7桁英数字で住所が変わる時代の到来

## 基本情報

- 作成日: 2025-05-27
- ステータス: [ ] アイデア段階 [ ] 調査中 [ ] 執筆中 [x] 完了

## 概要

2025年5月26日に開始された日本郵政の「デジタルアドレス」サービスを実際に使ってみた体験談と感想。7桁の英数字で住所を表現できる新しい仕組みの利便性と課題について、実際の使用感を交えて紹介する。

## 想定読者

- 新しいデジタルサービスに興味がある人
- 住所入力の手間を減らしたいと考えている人
- ECサイトやWebサービスを頻繁に利用する人
- 日本郵政のサービスを利用している人
- デジタル化の動向に関心がある人

## 主なポイント

1. **デジタルアドレスとは何か**
   - 7桁英数字で住所を表現する新システム
   - ゆうIDとの連携で無料取得可能
   - 引越し後も同じアドレスを継続使用可能

2. **実際の取得・設定体験**
   - 申請手順の簡単さ
   - ゆうIDの必要性
   - 設定時の注意点

3. **使ってみた感想**
   - 郵便局アプリでの送り状作成での利便性
   - 住所入力の時短効果
   - 部屋番号まで含む詳細住所の自動入力

4. **現時点での制限と課題**
   - デジタルアドレスのみでは郵便物を送れない制限
   - 対応サービスの少なさ
   - セキュリティ面での懸念（総当たり攻撃の可能性）
   - API利用は法人・個人事業主限定（個人開発者は利用不可）
   - 権限管理の課題：管理者のみがシステムリスト設定可能で実運用上不便

5. **今後の展望と期待**
   - 楽天やGMOなど大手企業の導入検討
   - タクシーやカーナビでの活用可能性
   - 10年構想での社会インフラ化
   - 個人開発者向けの利用プランの提供
   - 権限管理の改善（一般ユーザでもシステムリスト設定可能に）

6. **他のデジタル住所サービスとの比較**
   - 海外の類似サービスとの違い
   - 日本独自の特徴

7. **API利用の技術的詳細**
   - 事前準備：法人登録と管理画面でのシステムリスト登録
   - トークン取得からデジタルアドレス検索までの2段階認証
   - PHPでのサンプル実装
   - APIの利用制限と注意点
   - 開発者向けの実装ガイド
   - ドキュメント未記載の実装上の落とし穴（User-Agent必須など）
   - 権限管理の実運用上の課題

## 参考資料

- [日本郵便公式プレスリリース](https://www.post.japanpost.jp/notification/pressrelease/2025/00_honsha/0526_01.html)
- [デジタルアドレス公式サイト](https://lp.da.pf.japanpost.jp/)
- [郵便番号・デジタルアドレスAPI](https://guide-biz.da.pf.japanpost.jp/api/)
- [NHKニュース記事](https://www3.nhk.or.jp/news/html/20250527/k10014817321000.html)
- [Impress Watch記事](https://www.watch.impress.co.jp/docs/news/2017226.html)
- [Zenn技術記事](https://zenn.dev/matsubokkuri/articles/digital-address)

## 技術的詳細

### API利用手順

**事前準備:**
- 郵便番号・デジタルアドレス for Bizへの利用登録（法人・個人事業主限定）
  - 法人番号などの事業者情報が必要
  - 個人利用は不可
- 管理画面で「システムリスト」を登録
  - システム名の設定
  - URL（アクセス元のドメイン）の登録
  - IPアドレスの登録（最大10件まで）
  - **注意**: 管理者権限のみ設定可能（一般ユーザは不可）

1. **トークン取得API**にPOSTリクエストを送信
   - エンドポイント: `https://api.da.pf.japanpost.jp/api/v1/j/token`
   - 認証情報（client_id, secret_key）を含むJSONデータを送信
   - **注意**: User-Agentヘッダーの指定が実質必須（未指定だと403エラー）
   
2. **デジタルアドレス検索API**にGETリクエストを送信
   - エンドポイント: `https://api.da.pf.japanpost.jp/api/v1/searchcode/{デジタルアドレス}`
   - 取得したトークンをBearerトークンとして使用して住所情報を取得
   - User-Agentヘッダーの指定も推奨

### PHPサンプルコード（トークン取得）
```php
<?php
$url = 'https://api.da.pf.japanpost.jp/api/v1/j/token';
$data = [
  'grant_type' => 'client_credentials',
  'client_id' => '7d7eb07f4e2a4715a505f0ec2cad7f43',
  'secret_key' => '137f6a6842464a578fbf276a1245502d'
];

$ch = curl_init($url);
curl_setopt_array($ch, [
  CURLOPT_RETURNTRANSFER => true,
  CURLOPT_POST => true,
  CURLOPT_POSTFIELDS => json_encode($data),
  CURLOPT_HTTPHEADER => [
    'Content-Type: application/json',
    'User-Agent: Mozilla/5.0 (compatible; PHP cURL)',
    'X-Forwarded-For: ' . $_SERVER['REMOTE_ADDR']
  ],
  CURLOPT_HEADER => true,
  CURLOPT_VERBOSE => true
]);

$response = curl_exec($ch);
$http_code = curl_getinfo($ch, CURLINFO_HTTP_CODE);
$header_size = curl_getinfo($ch, CURLINFO_HEADER_SIZE);
$body = substr($response, $header_size);
$result = json_decode($body, true);

curl_close($ch);
?>
```

### PHPサンプルコード（住所検索）
```php
// トークンが取得できた場合、住所検索APIを呼び出す
if (isset($result['token'])) {
  $access_token = $result['token'];

  echo "\n\n=== 住所検索API呼び出し ===\n";

  // 住所検索API（例：FT7R9S6というデジタルアドレスで検索）
  $search_url = 'https://api.da.pf.japanpost.jp/api/v1/searchcode/FT7R9S6';

  $ch2 = curl_init($search_url);
  curl_setopt_array($ch2, [
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_HTTPHEADER => [
      'Authorization: Bearer ' . $access_token,
      'User-Agent: Mozilla/5.0 (compatible; PHP cURL)',
      'X-Forwarded-For: ' . $_SERVER['REMOTE_ADDR']
    ],
    CURLOPT_HEADER => true,
    CURLOPT_VERBOSE => true
  ]);

  $search_response = curl_exec($ch2);

  if ($search_response === false) {
    echo "住所検索cURLエラー: " . curl_error($ch2) . "\n";
  } else {
    $search_http_code = curl_getinfo($ch2, CURLINFO_HTTP_CODE);
    echo "住所検索HTTPステータスコード: " . $search_http_code . "\n\n";

    $search_header_size = curl_getinfo($ch2, CURLINFO_HEADER_SIZE);
    $search_headers = substr($search_response, 0, $search_header_size);
    $search_body = substr($search_response, $search_header_size);
    $search_result = json_decode($search_body, true);

    echo "=== 住所検索レスポンスヘッダー ===\n";
    echo $search_headers;
    echo "\n=== 住所検索レスポンスボディ ===\n";
    var_dump($search_result);
  }

  curl_close($ch2);
} else {
  echo "\nトークンの取得に失敗しました。\n";
}
?>
```

### APIレスポンス例

#### トークン取得APIのレスポンス
```php
array(4) {
  ["token"]=>
  string(619) "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJKUEQiLCJzdWIiOiJER0EgVE9LRU4iLCJzY29wZSI6IkoxIiwiY2xpZW50aWQiOiI3ZDdlYjA3ZjRlMmE0NzE1YTUwNWYwZWMyY2FkN2Y0MyIsImVjX2lkIjoiSlRFTkFOVC05NmRkM2U1NC0zMzgwLTQxYmMtYWIyOS1hZTU1YzllOTJkNzYiLCJpYXQiOjE3NDgzMTU2MDEsImV4cCI6MTc0ODMxNjIwMX0.KQhNYrO2Fm-jRlWmf3dKB-4CS_svKUz86widHRf69Ck9f2dq5vfkIgy5lnOEM0TvVXWKTDH9B67fnT1g1A4BcCRDd2HW8kMZSBkcL3Sdsmsm1tKDAgeSXGclA_wVCx3zZSCvbim_U1vmpAxZMM9SWOoeX45TYboCrBSkeIPvZ3tg_oqts4KerLgbT6G2PYi5mgMoC7pn_esq7EIuLd-3zyfzI0-pucygt5_zA1F9Sj3dqvXvMD60-WH5g8PyPE7tWwmDipjE77EnWtq5sQ6nMBnPrcQYwwLifbUaWcygoLP05UvUjcMuMC3iK2XuoEhD23fkC_nUNZAZMyjLA8helA"
  ["token_type"]=>
  string(3) "jwt"
  ["expires_in"]=>
  int(600)
  ["scope"]=>
  string(2) "J1"
}
```

**重要なポイント:**
- `token`: JWT形式のアクセストークン（619文字の長い文字列）
- `expires_in`: トークンの有効期限（600秒 = 10分）
- `token_type`: "jwt"固定
- `scope`: "J1"（一般向けAPIスコープ）

#### 住所検索APIのレスポンス（デジタルアドレス: FT7R9S6）
```php
array(5) {
  ["page"]=>
  int(1)
  ["limit"]=>
  int(1000)
  ["count"]=>
  int(1)
  ["searchtype"]=>
  string(7) "dgacode"
  ["addresses"]=>
  array(1) {
    [0]=>
    array(21) {
      ["dgacode"]=>
      string(7) "FT7R9S6"
      ["zip_code"]=>
      string(7) "2390806"
      ["pref_code"]=>
      string(2) "14"
      ["pref_name"]=>
      string(12) "神奈川県"
      ["city_name"]=>
      string(12) "横須賀市"
      ["town_name"]=>
      string(9) "池田町"
      ["block_name"]=>
      string(3) "1-1"
      ["other_name"]=>
      string(43) "四季の街パークヒルズ8番館507号"
      ["address"]=>
      string(79) "神奈川県横須賀市池田町1-1四季の街パークヒルズ8番館507号"
      ["longitude"]=>
      NULL
      ["latitude"]=>
      NULL
      // その他のフィールドは省略
    }
  }
}
```

**重要なポイント:**
- `dgacode`: 検索したデジタルアドレス
- `address`: 完全な住所文字列（部屋番号まで含む）
- `zip_code`: 郵便番号（7桁）
- `pref_code`: 都道府県コード（14 = 神奈川県）
- `longitude`, `latitude`: 座標情報（現在はNULL）
- 部屋番号まで詳細に取得可能（`other_name`フィールド）

## メモ

- サービス開始直後なので、実際の普及度や使い勝手の変化を継続的に観察する必要がある
- API利用規約が厳しく、個人開発者には使いにくい面もある
- API利用は2段階認証（トークン取得→検索）が必要で、トークンの有効期限管理が重要
- 住所検索APIではデジタルアドレス（例：FT7R9S6）をURLパスに含める形式
- **重要**: User-Agentヘッダーはドキュメントに必須と明記されていないが、指定しないと403エラーが発生する
- API利用前に「郵便番号・デジタルアドレス for Biz」管理画面でのシステムリスト登録が必須
- システムリスト登録項目：システム名、URL、IPアドレス（最大10件）
- 利用登録は法人・個人事業主限定で、法人番号等の事業者情報が必要
- 権限管理の課題：管理者のみがユーザ招待・システムリスト追加可能で、現場では管理者権限の共有が必要になる可能性
- 匿名性の課題：デジタルアドレスから住所が検索可能なため、フリマアプリなどでの匿名配送には使えない
- 総当たり攻撃への対策が利用規約レベルに留まっている点が気になる
- 将来的な法人向けサービスやブランドデジタルアドレス（例：JPP1234）の展開も興味深い
- 郵便番号制度との併存という位置づけ

## TODO

- [x] デジタルアドレスの取得を実際に試す
- [x] API利用手順の確認（トークン取得→検索の2段階）
- [x] PHPサンプルコードの取得
- [x] APIレスポンス構造の確認（トークン・住所検索両方）
- [ ] 郵便局アプリでの送り状作成を体験
- [x] デジタルアドレス検索APIの実際の動作確認
- [ ] 他の対応サービスが出てきたら試用
- [ ] セキュリティ面での詳細調査
- [ ] 海外の類似サービス（What3Words等）との比較調査
- [ ] 実際の普及状況の継続観察
- [ ] 企業向けAPI利用事例の調査
- [ ] 他言語（JavaScript、Python等）でのサンプル実装 