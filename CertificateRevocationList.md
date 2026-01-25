# 証明書失効リスト (Certificate Revocation List: CRL)

証明書失効リスト（CRL）は、失効したデジタル証明書のリストで、認証局（CA）によって定期的に更新・公開される。CRL は公開鍵インフラストラクチャ（PKI）の重要な部分であり、証明書の有効性を確認する際に不可欠である。

> 参照: [RFC 5280 Section 5](https://datatracker.ietf.org/doc/html/rfc5280#section-5) - X.509 PKI Certificate and CRL Profile

## CRL の主な目的

1. **証明書の失効情報の提供**: CRL は、盗難、キーの漏洩、所有者の情報変更、組織の閉鎖など、さまざまな理由で失効した証明書をリストする。
2. **セキュリティの維持**: 不正または誤って発行された証明書が利用されるのを防ぐ。

## CRL の構造

CRL は次の要素を含むことが一般的である。

- **バージョン**: CRL の形式のバージョン
- **発行者名**: CRL を発行した認証局の名前
- **この更新 (thisUpdate)**: CRL が最後に更新された日時
- **次の更新 (nextUpdate)**: CRL の次の予定更新日時
- **失効した証明書リスト**: 各証明書のシリアル番号と失効日が含まれる
- **署名**: 認証局が CRL を署名し、その完全性と真正性を保証する

## CRL の ASN.1 構造 (RFC 5280 Section 5.1)

RFC 5280 で定義される CRL のデータ構造を以下に示す。CRL は署名対象となる本体部分（TBSCertList）、署名アルゴリズム、署名値の 3 つの要素から構成される。

```asn1
CertificateList ::= SEQUENCE {
    tbsCertList          TBSCertList,
    signatureAlgorithm   AlgorithmIdentifier,
    signatureValue       BIT STRING
}

TBSCertList ::= SEQUENCE {
    version                 Version OPTIONAL,
                              -- if present, MUST be v2
    signature               AlgorithmIdentifier,
    issuer                  Name,
    thisUpdate              Time,
    nextUpdate              Time OPTIONAL,
    revokedCertificates     SEQUENCE OF SEQUENCE {
        userCertificate         CertificateSerialNumber,
        revocationDate          Time,
        crlEntryExtensions      Extensions OPTIONAL
                                  -- if present, MUST be v2
    } OPTIONAL,
    crlExtensions           [0] EXPLICIT Extensions OPTIONAL
                                  -- if present, MUST be v2
}
```

> 参照: [RFC 5280 Section 5.1](https://datatracker.ietf.org/doc/html/rfc5280#section-5.1)

## TBSCertList フィールド要件

TBSCertList（To Be Signed Certificate List）は CRL の署名対象となる本体部分である。各フィールドの RFC 5280 における要件を以下に示す。

| フィールド          | 説明                                | 要件                                   |
| ------------------- | ----------------------------------- | -------------------------------------- |
| version             | CRL のバージョン（拡張使用時は v2） | MUST（拡張使用時）                     |
| signature           | CRL 署名に使用するアルゴリズム      | MUST（signatureAlgorithm と一致）      |
| issuer              | CRL 発行者の識別名（DN）            | MUST（空でない X.500 DN）              |
| thisUpdate          | CRL が発行された日時                | MUST                                   |
| nextUpdate          | 次の CRL が発行される予定日時       | MUST（本プロファイル）                 |
| revokedCertificates | 失効した証明書のリスト              | OPTIONAL（失効証明書がない場合は省略） |
| crlExtensions       | CRL 拡張                            | OPTIONAL（バージョン 2 のみ）          |

### 時刻フィールドのエンコード要件

thisUpdate と nextUpdate の時刻フィールドは、年によって異なる形式でエンコードする必要がある。2050 年問題を考慮した規定となっている。

| 年          | 形式            | 要件 |
| ----------- | --------------- | ---- |
| 2049 年まで | UTCTime         | MUST |
| 2050 年以降 | GeneralizedTime | MUST |

> 参照: [RFC 5280 Section 5.1.2.4-5.1.2.5](https://datatracker.ietf.org/doc/html/rfc5280#section-5.1.2.4)

## CRL の利用

証明書を使用する際（例えば、SSL/TLS 通信、電子署名の検証など）、クライアントやサーバーは CRL を参照して、使用しようとしている証明書が失効していないことを確認する。CRL の確認は、証明書が依然として信頼できるかどうかを保証するために重要である。

## CRL のアクセス方法

CRL は、主に次のようなアクセス方法で利用される、

1. **CRL ダウンロード**:
   - 証明書には通常、CRL のダウンロード URL が含まれており、クライアントはこの URL から定期的に CRL をダウンロードしてローカルにキャッシュする
   - CRL は定期的に更新されるため、クライアントは更新間隔に従って新しいリストを取得する必要がある

2. **CRL 配布ポイント**:
   - 認証局は CRL を HTTP、FTP、LDAP などのプロトコルを使用して配布することがある
   - エンドユーザーやシステムは指定された配布ポイントから CRL を取得できる

3. **マニフェストとしての CRL**:
   - いくつかのシステムでは、CRL 自体を証明書の形式で配布し、その整合性を確保するためにデジタル署名がされる場合がある

## CRL 拡張 (RFC 5280 Section 5.2)

### 必須 CRL 拡張

RFC 5280 プロファイルに準拠する CRL 発行者は、以下の拡張を必ず含めなければならない。

| 拡張                     | OID       | 説明                 | 要件 |
| ------------------------ | --------- | -------------------- | ---- |
| Authority Key Identifier | 2.5.29.35 | CRL 署名鍵の識別子   | MUST |
| CRL Number               | 2.5.29.20 | CRL のシーケンス番号 | MUST |

### CRL Number 拡張

```asn1
id-ce-cRLNumber OBJECT IDENTIFIER ::= { id-ce 20 }

CRLNumber ::= INTEGER (0..MAX)
```

CRL Number は単調増加するシーケンス番号で、CRL の新旧を判断するために使用される。

> 参照: [RFC 5280 Section 5.2.3](https://datatracker.ietf.org/doc/html/rfc5280#section-5.2.3)

## CRL 配布ポイント (RFC 5280 Section 4.2.1.13)

証明書には CRL の取得先を示す配布ポイント（Distribution Point）を含めることができる。この拡張により、クライアントは証明書から直接 CRL の取得先 URL を知ることができる。

```asn1
id-ce-cRLDistributionPoints OBJECT IDENTIFIER ::= { id-ce 31 }

CRLDistributionPoints ::= SEQUENCE SIZE (1..MAX) OF DistributionPoint

DistributionPoint ::= SEQUENCE {
    distributionPoint       [0]     DistributionPointName OPTIONAL,
    reasons                 [1]     ReasonFlags OPTIONAL,
    cRLIssuer               [2]     GeneralNames OPTIONAL
}

DistributionPointName ::= CHOICE {
    fullName                [0]     GeneralNames,
    nameRelativeToCRLIssuer [1]     RelativeDistinguishedName
}
```

### ReasonFlags

配布ポイントでは、特定の失効理由のみを扱う CRL を指定できる。ReasonFlags は対象となる失効理由をビットフラグで表現する。

```asn1
ReasonFlags ::= BIT STRING {
    unused                  (0),
    keyCompromise           (1),
    cACompromise            (2),
    affiliationChanged      (3),
    superseded              (4),
    cessationOfOperation    (5),
    certificateHold         (6),
    privilegeWithdrawn      (7),
    aACompromise            (8)
}
```

> 参照: [RFC 5280 Section 4.2.1.13](https://datatracker.ietf.org/doc/html/rfc5280#section-4.2.1.13)

## Delta CRL (RFC 5280 Section 5.2.4)

Delta CRL は、完全な CRL への更新情報のみを含む差分 CRL である。

```asn1
id-ce-deltaCRLIndicator OBJECT IDENTIFIER ::= { id-ce 27 }

BaseCRLNumber ::= CRLNumber
```

### Delta CRL の利点

差分のみを配布する Delta CRL を使用することで、以下のような運用上の利点が得られる。

| 利点                   | 説明                        |
| ---------------------- | --------------------------- |
| ネットワーク負荷の軽減 | 完全な CRL より小さいサイズ |
| 処理時間の短縮         | 新しい失効情報のみを処理    |

### Delta CRL 結合条件

Delta CRL を完全な CRL と組み合わせて使用するためには、以下の条件をすべて満たす必要がある。

| 条件              | 説明                                   |
| ----------------- | -------------------------------------- |
| 同一発行者        | 完全な CRL と Delta CRL の発行者が同一 |
| 同一スコープ      | issuingDistributionPoint が同一        |
| CRL Number の関係 | 完全な CRL の番号 >= BaseCRLNumber     |
| 順序              | Delta CRL の番号 > 完全な CRL の番号   |

> 参照: [RFC 5280 Section 5.2.4](https://datatracker.ietf.org/doc/html/rfc5280#section-5.2.4)

## CRL エントリ拡張 (RFC 5280 Section 5.3)

失効した各証明書に対して追加情報を提供する拡張である。これらの拡張により、失効理由や無効となった日時などの詳細情報を含めることができる。

| 拡張               | OID       | 説明                                 |
| ------------------ | --------- | ------------------------------------ |
| Reason Code        | 2.5.29.21 | 失効理由                             |
| Invalidity Date    | 2.5.29.24 | 鍵の危殆化が発生したと推定される日時 |
| Certificate Issuer | 2.5.29.29 | 間接 CRL における証明書発行者        |

### 失効理由コード

証明書が失効された理由を示すコードである。失効理由によって、証明書の過去の有効性に関する解釈が異なる場合がある（例：鍵の危殆化の場合は過去の署名も信頼できない可能性がある）。

| コード               | 値  | 説明                             |
| -------------------- | --- | -------------------------------- |
| unspecified          | 0   | 理由不明                         |
| keyCompromise        | 1   | 鍵の危殆化                       |
| cACompromise         | 2   | CA の危殆化                      |
| affiliationChanged   | 3   | 所属の変更                       |
| superseded           | 4   | 置き換え                         |
| cessationOfOperation | 5   | 運用停止                         |
| certificateHold      | 6   | 一時停止（保留）                 |
| removeFromCRL        | 8   | CRL からの削除（Delta CRL のみ） |
| privilegeWithdrawn   | 9   | 権限の取り消し                   |
| aACompromise         | 10  | AA の危殆化                      |

> 参照: [RFC 5280 Section 5.3.1](https://datatracker.ietf.org/doc/html/rfc5280#section-5.3.1)

## CRL の課題と限界

- **スケーラビリティ**: 大規模な認証局で発行される証明書の数が多い場合、CRL のサイズが非常に大きくなり、ダウンロードと処理が煩雑になる。
- **更新間隔**: CRL の更新間隔が長い場合、最新の失効情報がすぐに反映されないため、セキュリティリスクが発生する可能性がある。

## OCSP (Online Certificate Status Protocol)

OCSP は CRL に代わるリアルタイムの証明書状態確認プロトコルである。CRL のタイムリネスの問題を解決するために使用される。OCSP を利用すると、クライアントは特定の証明書の有効性だけを認証局に問い合わせることができ、CRL 全体をダウンロードする必要がない。

> 参照: [RFC 6960](https://datatracker.ietf.org/doc/html/rfc6960) - X.509 OCSP

### OCSP の利点

OCSP は CRL と比較して以下のような利点を持つ。特に大規模環境やリアルタイム性が求められる場面で有効である。

| 利点             | 説明                                                         |
| ---------------- | ------------------------------------------------------------ |
| リアルタイム性   | 最新の失効情報をリアルタイムで確認できる                     |
| 帯域幅の節約     | 証明書のステータスのみを問い合わせるため、通信量が少ない     |
| スケーラビリティ | 大規模環境やモバイルデバイスに適しており、効率的な確認が可能 |

### 移行の現状

現在、インターネット上でのセキュアな通信を提供するために、特に SSL/TLS 証明書の管理において OCSP の利用が推進されている。多くの主要ブラウザやオペレーティングシステムは OCSP をサポートしており、リアルタイムでの証明書の有効性確認を行っている。特に Web セキュリティにおいては、ユーザーのセキュリティを確保するために、OCSP の利用が広がっている。

## OCSP リクエスト構造 (RFC 6960 Section 4.1.1)

OCSP リクエストは、確認したい証明書を特定するための情報を含む。CertID には証明書のシリアル番号と発行者情報のハッシュが含まれ、これにより対象の証明書を一意に識別する。

```asn1
OCSPRequest ::= SEQUENCE {
    tbsRequest                  TBSRequest,
    optionalSignature   [0]     EXPLICIT Signature OPTIONAL
}

TBSRequest ::= SEQUENCE {
    version             [0]     EXPLICIT Version DEFAULT v1,
    requestorName       [1]     EXPLICIT GeneralName OPTIONAL,
    requestList                 SEQUENCE OF Request,
    requestExtensions   [2]     EXPLICIT Extensions OPTIONAL
}

Request ::= SEQUENCE {
    reqCert                     CertID,
    singleRequestExtensions     [0] EXPLICIT Extensions OPTIONAL
}

CertID ::= SEQUENCE {
    hashAlgorithm       AlgorithmIdentifier,
    issuerNameHash      OCTET STRING,
    issuerKeyHash       OCTET STRING,
    serialNumber        CertificateSerialNumber
}
```

> 参照: [RFC 6960 Section 4.1.1](https://datatracker.ietf.org/doc/html/rfc6960#section-4.1.1)

## OCSP レスポンス構造 (RFC 6960 Section 4.2.1)

OCSP レスポンスは、リクエストの処理結果を示すステータスと、成功時には証明書の状態情報を含む。レスポンスは OCSP レスポンダによって署名され、その信頼性が保証される。

```asn1
OCSPResponse ::= SEQUENCE {
    responseStatus         OCSPResponseStatus,
    responseBytes          [0] EXPLICIT ResponseBytes OPTIONAL
}

OCSPResponseStatus ::= ENUMERATED {
    successful            (0),
    malformedRequest      (1),
    internalError         (2),
    tryLater              (3),
    -- (4) is not used
    sigRequired           (5),
    unauthorized          (6)
}

BasicOCSPResponse ::= SEQUENCE {
    tbsResponseData      ResponseData,
    signatureAlgorithm   AlgorithmIdentifier,
    signature            BIT STRING,
    certs            [0] EXPLICIT SEQUENCE OF Certificate OPTIONAL
}
```

### OCSP 証明書状態

OCSP レスポンスに含まれる証明書の状態は以下の 3 種類である。

| 状態    | 説明                                           |
| ------- | ---------------------------------------------- |
| good    | 証明書は失効していない                         |
| revoked | 証明書は失効している（または発行されていない） |
| unknown | レスポンダが証明書について知らない             |

> 参照: [RFC 6960 Section 4.2.1](https://datatracker.ietf.org/doc/html/rfc6960#section-4.2.1)

## OCSP レスポンス受理要件 (RFC 6960 Section 3.2)

クライアントは署名付きレスポンスを受理する前に以下を確認しなければならない（MUST）。

| 要件                | 説明                                               |
| ------------------- | -------------------------------------------------- |
| 証明書の一致        | レスポンスの証明書がリクエストと一致               |
| 署名の有効性        | レスポンスの署名が有効                             |
| 署名者の一致        | 署名者がリクエストの意図した受信者と一致           |
| 署名者の権限        | 署名者が当該証明書のレスポンスを提供する権限を持つ |
| thisUpdate の新しさ | ステータスが正しいとされる時刻が十分に最近         |
| nextUpdate の確認   | nextUpdate が現在時刻より後                        |

> 参照: [RFC 6960 Section 3.2](https://datatracker.ietf.org/doc/html/rfc6960#section-3.2)

## OCSP Stapling

OCSP Stapling は、サーバーが定期的に OCSP 応答を取得し、クライアントへの応答に含める技術である。これにより、クライアントが直接 OCSP サーバーに問い合わせる回数を減らすことができ、処理速度が向上し、プライバシーも向上する。

### OCSP Stapling の利点

OCSP Stapling を使用することで、通常の OCSP と比較して以下のような利点が得られる。

| 利点                  | 説明                                             |
| --------------------- | ------------------------------------------------ |
| OCSP サーバー負荷軽減 | クライアントが直接 OCSP サーバーに問い合わせない |
| 処理速度向上          | 追加のネットワークラウンドトリップが不要         |
| プライバシー向上      | クライアントの証明書確認が外部に漏れない         |

> 参照: [RFC 6066 Section 8](https://datatracker.ietf.org/doc/html/rfc6066#section-8) - TLS Extensions: Certificate Status Request

## CRL と OCSP の比較

CRL と OCSP はそれぞれ異なる特性を持つ。用途や環境に応じて適切な方式を選択、または併用することが重要である。

| 項目             | CRL                | OCSP                                     |
| ---------------- | ------------------ | ---------------------------------------- |
| 更新方式         | 定期的なリスト配布 | リアルタイム問い合わせ                   |
| データサイズ     | 大規模になりやすい | 個別証明書のステータスのみ               |
| タイムリネス     | 更新間隔に依存     | リアルタイム                             |
| ネットワーク負荷 | 高い（リスト全体） | 低い（個別問い合わせ）                   |
| プライバシー     | 高い               | 低い（問い合わせ先に証明書情報が漏れる） |
| オフライン利用   | 可能               | 不可                                     |

証明書失効リストは、セキュリティを維持するために重要な役割を果たすが、大規模なシステムやリアルタイム性が要求される場面では OCSP などの他の手法が補完的に使用されることが一般的である。

## 実装例 (OpenSSL)

### CRL のダウンロードと確認

```bash
# CRL のダウンロード
curl -o crl.der http://example.com/crl.der

# CRL の内容表示
openssl crl -in crl.der -inform DER -text -noout

# CRL による証明書検証
openssl verify -crl_check -CAfile ca.pem -CRLfile crl.pem cert.pem
```

### OCSP による証明書確認

```bash
# OCSP リクエストの送信
openssl ocsp -issuer issuer.pem -cert cert.pem \
    -url http://ocsp.example.com -resp_text

# OCSP レスポンスの検証
openssl ocsp -issuer issuer.pem -cert cert.pem \
    -url http://ocsp.example.com -VAfile va.pem
```

## 関連 RFC

CRL および OCSP に関連する主要な RFC 文書を以下に示す。

| RFC                                                       | タイトル                              | 説明                          |
| --------------------------------------------------------- | ------------------------------------- | ----------------------------- |
| [RFC 5280](https://datatracker.ietf.org/doc/html/rfc5280) | X.509 PKI Certificate and CRL Profile | CRL プロファイル（Section 5） |
| [RFC 6960](https://datatracker.ietf.org/doc/html/rfc6960) | X.509 OCSP                            | OCSP プロトコル               |
| [RFC 5019](https://datatracker.ietf.org/doc/html/rfc5019) | Lightweight OCSP Profile              | 軽量 OCSP プロファイル        |
| [RFC 6066](https://datatracker.ietf.org/doc/html/rfc6066) | TLS Extensions                        | OCSP Stapling                 |
| [RFC 2119](https://datatracker.ietf.org/doc/html/rfc2119) | Key words for RFCs                    | MUST/SHOULD/MAY の定義        |

## 関連法令

### 電子署名法（平成十二年法律第百二号）

電子署名及び認証業務に関する法律において、電子署名の定義と法的効力が規定されている。

> 詳細: [e-Gov 法令検索 - 電子署名法](https://laws.e-gov.go.jp/law/412AC0000000102)
