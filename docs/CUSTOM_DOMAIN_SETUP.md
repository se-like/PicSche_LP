# GitHub Pages で cloudcrowd.cloud を設定する手順

ドメイン **cloudcrowd.cloud** を PicSche LP の GitHub Pages に設定する手順です。

---

## 前提

- リポジトリ: **se-like/PicSche_LP**
- 現在の Pages URL: `https://se-like.github.io/PicSche_LP/`
- 設定後: `https://cloudcrowd.cloud/` または `https://www.cloudcrowd.cloud/` で表示

**重要:** 公式の推奨は「**先に GitHub でカスタムドメインを保存してから、DNS を設定する**」です。順序を逆にすると、第三者にサブドメインを奪われるリスクがあります。

---

## ステップ 1: GitHub でカスタムドメインを登録

1. **GitHub** で `se-like/PicSche_LP` を開く
2. **Settings** → 左メニュー **Pages**
3. **Custom domain** の入力欄に次のどちらかを入力して **Save**
   - ルートで使う場合: `cloudcrowd.cloud`
   - www で使う場合: `www.cloudcrowd.cloud`
4. （後で HTTPS を有効にするため）ここでは **Enforce HTTPS はまだオフ**のままでよい
5. ブランチ公開の場合は、リポジトリに **CNAME ファイルが自動作成**される（中身は入力したドメイン名）

※ ルート（apex）と www の両方を使う場合は、**どちらか一方**をここで設定。もう一方は DNS で設定し、GitHub がリダイレクトします。

---

## ステップ 2: DNS プロバイダでレコードを追加

ドメインの管理画面（お名前.com、Cloudflare、Google Domains、AWS Route53 など）で、次のように設定します。

### パターン A: ルートドメイン（cloudcrowd.cloud）で使う

**A レコードを 4 本追加**（ホストは `@` または空、値は以下の 4 つ）

| タイプ | ホスト / 名前 | 値 | TTL（任意） |
|--------|----------------|-----|-------------|
| A | @ | 185.199.108.153 | 3600 |
| A | @ | 185.199.109.153 | 3600 |
| A | @ | 185.199.110.153 | 3600 |
| A | @ | 185.199.111.153 | 3600 |

**IPv6 も使う場合（任意）** — AAAA レコードを 4 本

| タイプ | ホスト / 名前 | 値 | TTL（任意） |
|--------|----------------|-----|-------------|
| AAAA | @ | 2606:50c0:8000::153 | 3600 |
| AAAA | @ | 2606:50c0:8001::153 | 3600 |
| AAAA | @ | 2606:50c0:8002::153 | 3600 |
| AAAA | @ | 2606:50c0:8003::153 | 3600 |

**ALIAS / ANAME が使える場合**（Cloudflare、DNSimple など）  
- ホスト: `@`  
- 値: `se-like.github.io`  
- A レコード 4 本の代わりにこれ 1 本で可。

---

### パターン B: www（www.cloudcrowd.cloud）で使う

**CNAME レコードを 1 本追加**

| タイプ | ホスト / 名前 | 値 | TTL（任意） |
|--------|----------------|-----|-------------|
| CNAME | www | se-like.github.io | 3600 |

※ 値は **リポジトリ名を含めず** `se-like.github.io` だけにします。

---

### パターン C: ルートと www の両方で使う（推奨）

1. **ステップ 1** では、どちらか一方（例: `cloudcrowd.cloud`）だけを Custom domain に保存
2. **DNS は両方**設定する  
   - ルート: 上記「パターン A」の A レコード（または ALIAS/ANAME）  
   - www: 上記「パターン B」の CNAME
3. すると GitHub が自動で、  
   - `cloudcrowd.cloud` ⇔ `www.cloudcrowd.cloud` のリダイレクトをしてくれます

---

## ステップ 3: 反映待ちと確認

- DNS の反映は **数分〜最大 24 時間**かかることがあります
- 確認例（ルートで A レコードを設定した場合）:

```bash
# A レコード
dig cloudcrowd.cloud +noall +answer -t A

# www で CNAME を設定した場合
dig www.cloudcrowd.cloud +noall +answer
```

- GitHub の **Settings → Pages** で、Custom domain の横に「DNS check successful」などと出れば OK

---

## ステップ 4: HTTPS（Enforce HTTPS）を有効にする

1. DNS が正しく反映し、Pages のカスタムドメインが「チェック成功」になったら
2. **Settings → Pages** の **Enforce HTTPS** にチェックを入れる
3. 証明書発行まで **最大 24 時間**かかることがあるので、すぐにオンにできない場合は時間をおいて再度試す

---

## よくあるトラブル

| 現象 | 対処 |
|------|------|
| 「DNS check failed」のまま | レコードのホスト名・値が公式のとおりか確認。CNAME の値にリポジトリ名が入っていないか確認 |
| HTTPS が有効にならない | DNS 反映と証明書発行を待つ。一度 Custom domain を空にして Save し、再度ドメインを入力して Save すると証明書が再発行されることがある |
| www とルートで二重に設定したい | GitHub の Custom domain には **1 つだけ**入力。もう一方は DNS だけで設定する |
| 既存の A レコードや CNAME がある | ルート（@）向けのデフォルトの A/CNAME は削除または変更し、上記の値に統一する |

---

## 参考リンク

- [Managing a custom domain for your GitHub Pages site](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site)
- [Troubleshooting custom domains and GitHub Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/troubleshooting-custom-domains-and-github-pages)
