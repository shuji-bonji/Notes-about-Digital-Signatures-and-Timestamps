# JWS (JSON Web Signature)

JWS（JSON Web Signature）は、JSON ベースのデータ構造を使用してデジタル署名または MAC（Message Authentication Code）でコンテンツを保護するための標準仕様である。JWS はデータの完全性と認証を保証するために使用される。

> 参照: [RFC 7515](https://datatracker.ietf.org/doc/html/rfc7515) - JSON Web Signature (JWS)

## 概要

JWS は任意のコンテンツ（ペイロード）に対して署名を行い、そのコンテンツが改ざんされていないことを検証可能にする。JWT（JSON Web Token）の署名部分は JWS の仕様に基づいている。

### JWS の主な用途

JWS は以下のような用途で使用される。署名によりデータの完全性と発信者の認証を保証する。

| 用途               | 説明                                                           |
| ------------------ | -------------------------------------------------------------- |
| データの完全性保証 | コンテンツが改ざんされていないことを検証                       |
| 発信者の認証       | 署名者が正当な発信者であることを確認                           |
| 否認防止           | 非対称鍵アルゴリズムを使用した場合、署名者は署名を否認できない |

## JWT と JWS の違い

JWT と JWS のデータの構造には基本的な違いはなく、ペイロードに含まれる情報の種類と目的によって使い分けられている。

### ペイロードに含まれる情報の種類と目的の違い

JWT と JWS はペイロードに含める情報の種類と目的が異なる。JWT はクレームベースの認証トークンであるのに対し、JWS は汎用的な署名フォーマットである。

| 項目             | JWT                                                                | JWS                                      |
| ---------------- | ------------------------------------------------------------------ | ---------------------------------------- |
| ペイロードの内容 | クレーム（Claims）- ユーザー情報や認可情報などの JSON オブジェクト | 任意のコンテンツ（バイナリデータも可能） |
| 主な目的         | 認証・認可のためのトークン                                         | 任意のデータへの署名                     |
| 仕様の関係       | JWS を使用して署名される                                           | JWT の署名メカニズムを提供               |

### 構造上の主な違い

データ構造の観点から見た JWT と JWS の違いを以下に示す。

| 項目             | JWT                               | JWS                      |
| ---------------- | --------------------------------- | ------------------------ |
| ペイロードの形式 | MUST be JSON (Claims Set)         | 任意のオクテット列       |
| 登録済みクレーム | iss, sub, aud, exp, nbf, iat, jti | なし（ペイロードに依存） |
| typ ヘッダー     | "JWT" を推奨                      | アプリケーション依存     |

## JWS の構造

JWS は JOSE Header、JWS Payload、JWS Signature の 3 つの部分から構成される。

```
BASE64URL(UTF8(JOSE Header)).BASE64URL(JWS Payload).BASE64URL(JWS Signature)
```

### 各要素の説明

JWS を構成する 3 つの要素の詳細を以下に示す。

| 要素          | 説明                                                         | RFC 参照           |
| ------------- | ------------------------------------------------------------ | ------------------ |
| JOSE Header   | 署名アルゴリズムやその他のメタデータを含む JSON オブジェクト | RFC 7515 Section 4 |
| JWS Payload   | 署名対象のコンテンツ（任意のオクテット列）                   | RFC 7515 Section 3 |
| JWS Signature | JOSE Header と JWS Payload に対する署名値                    | RFC 7515 Section 3 |

> 参照: [RFC 7515 Section 3](https://datatracker.ietf.org/doc/html/rfc7515#section-3) - JSON Web Signature (JWS) Overview

## JOSE Header (RFC 7515 Section 4)

JOSE Header は署名または MAC アルゴリズムとパラメータを記述する JSON オブジェクトである。

### 登録済みヘッダーパラメータ

JOSE Header で使用できる登録済みパラメータを以下に示す。alg は必須であり、その他のパラメータは鍵の識別や証明書の参照に使用される。

| パラメータ | 説明                                  | 要件     | RFC 参照       |
| ---------- | ------------------------------------- | -------- | -------------- |
| alg        | 署名または MAC に使用するアルゴリズム | MUST     | Section 4.1.1  |
| jku        | JWK Set の URL                        | OPTIONAL | Section 4.1.2  |
| jwk        | JSON Web Key（公開鍵）                | OPTIONAL | Section 4.1.3  |
| kid        | 鍵の識別子                            | OPTIONAL | Section 4.1.4  |
| x5u        | X.509 証明書の URL                    | OPTIONAL | Section 4.1.5  |
| x5c        | X.509 証明書チェーン                  | OPTIONAL | Section 4.1.6  |
| x5t        | X.509 証明書の SHA-1 サムプリント     | OPTIONAL | Section 4.1.7  |
| x5t#S256   | X.509 証明書の SHA-256 サムプリント   | OPTIONAL | Section 4.1.8  |
| typ        | トークンのタイプ                      | OPTIONAL | Section 4.1.9  |
| cty        | コンテンツタイプ                      | OPTIONAL | Section 4.1.10 |
| crit       | クリティカル拡張                      | OPTIONAL | Section 4.1.11 |

### ヘッダーの例

```json
{
	"alg": "RS256",
	"typ": "JWS",
	"kid": "2024-01-key"
}
```

> 参照: [RFC 7515 Section 4.1](https://datatracker.ietf.org/doc/html/rfc7515#section-4.1)

## シリアライゼーション形式 (RFC 7515 Section 7)

JWS には 2 つのシリアライゼーション形式がある。

### JWS Compact Serialization

URL セーフなコンパクトな文字列表現。3 つの Base64URL エンコードされた部分をピリオドで連結する。

```
eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ.cC4hiUPoj9Eetdgtv3hF80EGrhuB__dzERat0XF9g2VtQgr9PJbu3XOiZj5RZmh7AAuHIm4Bh-0Qc_lF5YKt_O8W2Fp5jujGbds9uJdbF9CUAr7t1dnZcAcQjbKBYNX4BAynRFdiuB--f_nZLgrnbyTyWzO75vRK5h6xBArLIARNPvkSjtQBMHlb1L07Qe7K0GarZRmB_eSN9383LcOLn6_dO--xi12jzDwusC-eOkHWEsqtFZESc6BfI7noOPqvhJ1phCnvWh6IeYI2w9QOYEUipUTI8np6LbgGY9Fs98rqVt5AXLIhWkWywlVmtVrBp0igcN_IoypGlUPQGe77Rw
```

### JWS JSON Serialization

より詳細な JSON 表現。複数の署名をサポートする。

#### General JWS JSON Serialization

```json
{
	"payload": "eyJpc3MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ",
	"signatures": [
		{
			"protected": "eyJhbGciOiJSUzI1NiJ9",
			"header": {
				"kid": "2024-01-key"
			},
			"signature": "cC4hiUPoj9Eetdgtv3hF80EGrhuB..."
		}
	]
}
```

#### Flattened JWS JSON Serialization

単一の署名の場合に使用する簡略化された形式。

```json
{
	"payload": "eyJpc3MiOiJqb2UiLA0KICJleHAiOjEzMDA4MTkzODAsDQogImh0dHA6Ly9leGFtcGxlLmNvbS9pc19yb290Ijp0cnVlfQ",
	"protected": "eyJhbGciOiJSUzI1NiJ9",
	"header": {
		"kid": "2024-01-key"
	},
	"signature": "cC4hiUPoj9Eetdgtv3hF80EGrhuB..."
}
```

> 参照: [RFC 7515 Section 7](https://datatracker.ietf.org/doc/html/rfc7515#section-7)

## 署名アルゴリズム (RFC 7518)

### デジタル署名アルゴリズム

非対称鍵を使用するデジタル署名アルゴリズムを以下に示す。否認防止が必要な場合はこれらのアルゴリズムを使用する。

| アルゴリズム | 説明                            | 要件        |
| ------------ | ------------------------------- | ----------- |
| RS256        | RSASSA-PKCS1-v1_5 using SHA-256 | RECOMMENDED |
| RS384        | RSASSA-PKCS1-v1_5 using SHA-384 | OPTIONAL    |
| RS512        | RSASSA-PKCS1-v1_5 using SHA-512 | OPTIONAL    |
| ES256        | ECDSA using P-256 and SHA-256   | RECOMMENDED |
| ES384        | ECDSA using P-384 and SHA-384   | OPTIONAL    |
| ES512        | ECDSA using P-521 and SHA-512   | OPTIONAL    |
| PS256        | RSASSA-PSS using SHA-256        | OPTIONAL    |
| PS384        | RSASSA-PSS using SHA-384        | OPTIONAL    |
| PS512        | RSASSA-PSS using SHA-512        | OPTIONAL    |

### MAC アルゴリズム

対称鍵を使用する MAC（メッセージ認証コード）アルゴリズムを以下に示す。送信者と受信者が同じ秘密鍵を共有する場合に使用する。

| アルゴリズム | 説明               | 要件     |
| ------------ | ------------------ | -------- |
| HS256        | HMAC using SHA-256 | REQUIRED |
| HS384        | HMAC using SHA-384 | OPTIONAL |
| HS512        | HMAC using SHA-512 | OPTIONAL |

### アルゴリズムなし

署名を行わない Unsecured JWS も仕様上は定義されているが、セキュリティ上の理由から本番環境での使用は避けるべきである。

| アルゴリズム | 説明     | 要件                      |
| ------------ | -------- | ------------------------- |
| none         | 署名なし | MAY（本番環境では非推奨） |

> 参照: [RFC 7518](https://datatracker.ietf.org/doc/html/rfc7518) - JSON Web Algorithms (JWA)

## 署名の生成 (RFC 7515 Section 5.1)

### 署名生成手順

RFC 7515 で定義される JWS の署名生成手順を以下に示す。

| ステップ | 説明                                                             | 要件 |
| -------- | ---------------------------------------------------------------- | ---- | --- | --- | ----------------------- | --- |
| 1        | JWS Payload となるコンテンツを作成                               | -    |
| 2        | BASE64URL(JWS Payload) を計算                                    | -    |
| 3        | JOSE Header を含む JSON オブジェクトを作成                       | -    |
| 4        | BASE64URL(UTF8(JOSE Header)) を計算                              | -    |
| 5        | JWS Signing Input を生成: `BASE64URL(UTF8(JWS Protected Header)) |      | '.' |     | BASE64URL(JWS Payload)` | -   |
| 6        | JWS Signing Input に対して署名または MAC を計算                  | -    |
| 7        | BASE64URL(JWS Signature) を計算                                  | -    |
| 8        | シリアライゼーション形式に従って出力を生成                       | -    |

### 署名生成のフロー

```mermaid
flowchart TD
    A[JWS Payload] --> B[BASE64URL エンコード]
    C[JOSE Header] --> D[UTF-8 変換]
    D --> E[BASE64URL エンコード]
    B --> F[JWS Signing Input 生成]
    E --> F
    F --> G[Header.Payload を連結]
    G --> H[署名アルゴリズム適用]
    I[秘密鍵] --> H
    H --> J[JWS Signature]
    J --> K[BASE64URL エンコード]
    K --> L[JWS 完成]
```

> 参照: [RFC 7515 Section 5.1](https://datatracker.ietf.org/doc/html/rfc7515#section-5.1)

## 署名の検証 (RFC 7515 Section 5.2)

### 署名検証手順

受信した JWS を検証する際の手順を以下に示す。検証に失敗した場合は JWS を無効として拒否する必要がある。

| ステップ | 説明                                                    | 要件 |
| -------- | ------------------------------------------------------- | ---- |
| 1        | JWS 表現をパースして各要素を抽出                        | -    |
| 2        | JOSE Header を Base64URL デコード                       | MUST |
| 3        | 結果が UTF-8 エンコードされた JSON であることを確認     | MUST |
| 4        | JOSE Header を構築（Protected と Unprotected をマージ） | -    |
| 5        | 全てのパラメータが理解可能であることを確認              | MUST |
| 6        | JWS Payload を Base64URL デコード                       | MUST |
| 7        | JWS Signature を Base64URL デコード                     | MUST |
| 8        | JWS Signing Input に対して署名を検証                    | MUST |
| 9        | 複数署名の場合、各署名に対して処理を繰り返す            | -    |
| 10       | 検証が成功しなければ JWS は無効として拒否               | MUST |

> 参照: [RFC 7515 Section 5.2](https://datatracker.ietf.org/doc/html/rfc7515#section-5.2)

## 鍵の識別 (RFC 7515 Section 6)

JWS を検証する際、正しい鍵を特定するために以下のヘッダーパラメータが使用される。複数の鍵が存在する環境では、これらのパラメータにより適切な鍵を選択できる。

| パラメータ   | 説明                 | 使用場面                   |
| ------------ | -------------------- | -------------------------- |
| kid          | 鍵の識別子           | 複数の鍵から適切な鍵を選択 |
| jku          | JWK Set の URL       | 動的に鍵を取得             |
| jwk          | インライン JWK       | 鍵を直接提供               |
| x5u          | X.509 証明書 URL     | X.509 証明書から鍵を取得   |
| x5c          | X.509 証明書チェーン | 証明書チェーンから鍵を取得 |
| x5t/x5t#S256 | 証明書のサムプリント | 証明書を識別               |

> 参照: [RFC 7515 Section 6](https://datatracker.ietf.org/doc/html/rfc7515#section-6)

## セキュリティ上の考慮事項 (RFC 7515 Section 10)

### 主なセキュリティ上の注意点

JWS を安全に実装・運用するために考慮すべきセキュリティ上の注意点を以下に示す。

| 項目              | 説明                                     | RFC 参照      |
| ----------------- | ---------------------------------------- | ------------- |
| 鍵のエントロピー  | 十分なエントロピーを持つ鍵を使用すること | Section 10.1  |
| 鍵の保護          | 秘密鍵は適切に保護すること               | Section 10.2  |
| 鍵の出所認証      | 鍵の出所を確認すること                   | Section 10.3  |
| 暗号の俊敏性      | アルゴリズムの移行が可能な設計にすること | Section 10.4  |
| 署名と MAC の違い | 否認防止には署名を使用（MAC は不可）     | Section 10.5  |
| アルゴリズム検証  | 予期しないアルゴリズムを拒否すること     | Section 10.6  |
| アルゴリズム保護  | "alg" の改ざんを防ぐこと                 | Section 10.7  |
| リプレイ攻撃対策  | 必要に応じてリプレイ攻撃を防ぐこと       | Section 10.10 |

### デジタル署名と MAC の違い

デジタル署名と MAC は両方とも完全性を保証するが、鍵の種類と否認防止の特性が異なる。用途に応じて適切な方式を選択する必要がある。

| 項目     | デジタル署名                       | MAC                                  |
| -------- | ---------------------------------- | ------------------------------------ |
| 鍵の種類 | 非対称鍵（公開鍵/秘密鍵）          | 対称鍵（共有秘密）                   |
| 否認防止 | 可能（秘密鍵保持者のみが署名可能） | 不可（共有秘密を知る全員が生成可能） |
| 検証者   | 公開鍵を持つ誰でも                 | 共有秘密を持つ者のみ                 |
| 使用場面 | 第三者への証明が必要な場合         | 当事者間の認証                       |

> 参照: [RFC 7515 Section 10](https://datatracker.ietf.org/doc/html/rfc7515#section-10)

## 実装例

### Node.js での JWS 生成（RS256）

```javascript
const crypto = require('crypto');

// JOSE Header
const header = { alg: 'RS256', typ: 'JWS' };
const encodedHeader = Buffer.from(JSON.stringify(header)).toString('base64url');

// Payload
const payload = { data: 'Hello, World!' };
const encodedPayload = Buffer.from(JSON.stringify(payload)).toString('base64url');

// Signing Input
const signingInput = `${encodedHeader}.${encodedPayload}`;

// Sign
const privateKey = fs.readFileSync('private.pem');
const sign = crypto.createSign('RSA-SHA256');
sign.update(signingInput);
const signature = sign.sign(privateKey, 'base64url');

// JWS
const jws = `${signingInput}.${signature}`;
```

### OpenSSL での署名検証

```bash
# JWS から署名部分を抽出してデコード
echo -n '<signature>' | base64 -D > signature.bin

# Signing Input を作成
echo -n '<header>.<payload>' > signing_input.txt

# 署名を検証
openssl dgst -sha256 -verify public.pem -signature signature.bin signing_input.txt
```

## 関連 RFC

JWS およびその関連技術に関する RFC 文書を以下に示す。

| RFC                                                       | タイトル                   | 説明                             |
| --------------------------------------------------------- | -------------------------- | -------------------------------- |
| [RFC 7515](https://datatracker.ietf.org/doc/html/rfc7515) | JSON Web Signature (JWS)   | 本文書の主要参照                 |
| [RFC 7516](https://datatracker.ietf.org/doc/html/rfc7516) | JSON Web Encryption (JWE)  | 暗号化仕様                       |
| [RFC 7517](https://datatracker.ietf.org/doc/html/rfc7517) | JSON Web Key (JWK)         | 鍵表現仕様                       |
| [RFC 7518](https://datatracker.ietf.org/doc/html/rfc7518) | JSON Web Algorithms (JWA)  | 暗号アルゴリズム仕様             |
| [RFC 7519](https://datatracker.ietf.org/doc/html/rfc7519) | JSON Web Token (JWT)       | JWT 仕様（JWS を使用）           |
| [RFC 8725](https://datatracker.ietf.org/doc/html/rfc8725) | JWT Best Current Practices | セキュリティのベストプラクティス |
| [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) | Key words for RFCs         | MUST/SHOULD/MAY の定義           |

## 参考資料

- [IANA JSON Web Signature and Encryption Header Parameters](https://www.iana.org/assignments/jose/jose.xhtml) - 登録済みヘッダーパラメータ一覧
- [jwt.io](https://jwt.io) - JWT/JWS のデバッグツール
