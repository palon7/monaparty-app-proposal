<pre>
id:1 
title: NFT Image Specification
author: Ryota Uno (@palon7)
status: Draft
type: Standards Track
created: 2021-03-29
</pre>

# NFT 画像の規格

## 注意

本提案はドラフト段階です。議論や意見を広く歓迎します。

## 概要

本文書では、Monaparty 上で発行されるアセットの description を利用したメタデータ規格について提案する。
この提案ではアセットと画像データを紐付け、表示するための規格を提供する。

本提案では画像データの永続性、信頼性等を確保する方法については言及しない。

## 背景

NFT アートの人気に見られるように、画像と紐付くトークンはトークンプラットフォームにおける重要な存在である。しかし現在 Monaparty には、自由な形の画像データをトークンに紐付けられる規格が存在しない。自由なメタデータを持たせる仕組みとしては[XMPIP-0019](https://github.com/monaparty/XMPIP/blob/master/XMPIP-0019.md)が計画されているが、現時点では実装されていない。

一方、[Monacard](https://card.mona.jp/)はアセットの**description**欄に JSON を記載するという方法でメタデータを記録した。この方法だと発行時のトランザクションのサイズが肥大化することになるが、現状の Monacoin の手数料水準では問題ないと考えられる。
そこで、同手法を用いて Monaparty アセットに自由な形の画像を紐付ける規格を提案する。

## 用語

本文書中のキーワード`MUST, MUST NOT, REQUIRED, SHALL, SHALL NOT, SHOULD, SHOULD NOT, RECOMMENDED, MAY, OPTIONAL`は RFC 2119 に沿って解釈される。

## 仕様

本規格に準拠するアセットは、以下の JSON Schema に準拠した JSON をアセットの**description**として記載しなくてはならない **(MUST)** 。JSON には本規格にはないプロパティを記載してもよい **(MAY)**。

アセットの**description**には JSON として解析できない文字列を記載してはならない **(MUST NOT)** 。

### JSON Schema

```json
{
  "title": "Monaparty Image NFT Metadata",
  "type": "object",
  "properties": {
    "name": {
      "type": "string"
    },
    "description": {
      "type": "string"
    },
    "image": {
      "type": "string"
    }
  },
  "required": ["name", "image"]
}
```

### 実際の JSON の例

```json
{
  "name": "モナコインちゃん",
  "description": "モナコインちゃんのNFTです。",
  "image": "bafkreifzjut3te2nhyekklss27nh3k72ysco7y32koao5eei66wof36n5e"
}
```

### name

アセットを識別する名前。

### description

アセットの説明文。この項目は省略してもよい。 **(OPTIONAL)**

### image

アセットと紐付く画像ファイルの IPFS 上の cid。バージョンは v1 を使用したほうがよい **(SHOULD)**。

画像ファイルの MIME type は`image/*`でなければならない **(MUST)**。`image/png`, `image/apng`, `image/jpeg`, `image/gif`形式を使用した方がよい **(SHOULD)**。それ以外の形式の場合、アプリケーションは表示を拒否してもよい **(MAY)**。

画像のサイズは短辺を 300px 以上、かつ長辺を 1920px 以下にしたほうがよい **(SHOULD)**。

## 理論的根拠

### IPFS

本規格では画像の保存先を IPFS に限定している。これは、IPFS には次の利点があるためである。

- 分散型システムであるため、一つのサーバ/個人/企業に依存する必要がない。
- データの保持には pinning が必要だが、これはアセットの発行者だけではなく誰でも行うことができる。
- IPFS のハッシュ値はファイルに依存しているため、オフチェーンで画像データを書き換えることは不可能である。
- 一度 IPFS 上からデータが失われたとしても、元のファイルがあれば再アップロードすることで再度アセットと画像の紐付けを証明できる。

### 画像のファイル形式

本規格は一般的な画像ファイルを想定しているため、MIME type を`image/*`に限定した。また、`image/png`, `image/gif`, `image/jpeg`はインターネット上で広く使われており、`image/apng`は高画質のアニメーションに対応した形式の中で多くの環境に対応しているため、これらを推奨する形式とした。

## 懸念点

### IPFS 上のファイルの消失リスク

本規格では画像の保存先を IPFS 上に限定しているが、IPFS 上のファイルは適切に pinning されない限り消滅してしまうリスクが高い。

もっとも、Ethereum などの NFT でも共通する問題であり、IPFS 以外に保存した場合でも発生しうるリスクではある。

### 細工された画像ファイルによるリスク

SVG ファイルには Javascript を埋め込むことができ、ユーザーが直接 SVG ファイルを開いた場合 XSS が発生するリスクがある。
アプリケーションと同一のオリジンからその画像が提供されていた場合、機密データ（cookie など）が漏洩する可能性がある。

本規格に沿った実装をする場合は「svg 形式の画像を不用意に同一 URL から提供しない」「svg 形式の画像を表示しない」「ユーザに注意喚起する」などの対応をする必要があると考えられる。

参考: v4.0.3-5.2.7 in [OWASP ASVS 4.0](https://github.com/OWASP/ASVS/blob/v4.0.3/4.0/en/0x13-V5-Validation-Sanitization-Encoding.md)

### description の書き換えリスク

現在、アセットの description は owner なら`issuance`メッセージを使っていつでも書き換えることができる。
書き換えた履歴はチェーン上に残るが、単純な実装ではこれを検知できず、持っているアセットの画像が書き換えられたように見える可能性がある。

これを防ぐためには、以下のような対策が考えられる。

- issuer は価値の証明のため、事前に所有権を Burn アドレス（`MMonapartyMMMMMMMMMMMMMMMMMMMUzGgh`など）に transfer しておく。
- アプリケーションは画像データを参照する際にアセットの**issuance**の履歴を検索し、description 内の`image`が書き換えられていないかを確認する。

## 著作権

本文書は[CC0](https://creativecommons.org/publicdomain/zero/1.0/)でライセンスされる。
