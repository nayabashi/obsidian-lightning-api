# ⚡️ n8n ワークフロー セットアップガイド

## 📋 必要な準備

### 1. GitHub Personal Access Token (PAT) の作成

2つのトークンが必要です：

#### Token 1: Obsidianリポジトリ読み取り用（プライベート）
- **Scope**: `repo` (Full control of private repositories)
- **用途**: `nayabashi/Obsidian` からファイルを読み取る

#### Token 2: APIリポジトリ書き込み用（公開）
- **Scope**: `public_repo` (Access public repositories)
- **用途**: `nayabashi/obsidian-lightning-api` に index.json を書き込む

### トークン作成手順
1. GitHub > Settings > Developer settings > Personal access tokens > Tokens (classic)
2. Generate new token (classic)
3. Note: `n8n Obsidian Read` / `n8n Lightning API Write`
4. スコープを選択して Generate
5. トークンをコピー（再表示不可）

---

## 🔧 n8n セットアップ手順

### Step 1: Credentialの追加

#### Credential 1: GitHub PAT (Obsidian)
1. n8n > Credentials > Add Credential
2. タイプ: **Header Auth**
3. 設定:
   - Name: `GitHub PAT (Obsidian)`
   - Header Name: `Authorization`
   - Header Value: `token YOUR_OBSIDIAN_READ_TOKEN`

#### Credential 2: GitHub PAT (API)
1. n8n > Credentials > Add Credential
2. タイプ: **Header Auth**
3. 設定:
   - Name: `GitHub PAT (API repo)`
   - Header Name: `Authorization`
   - Header Value: `token YOUR_API_WRITE_TOKEN`

### Step 2: ワークフローのインポート

1. n8n > Workflows > Import from File
2. `n8n-workflow-obsidian-lightning.json` を選択
3. インポート完了

### Step 3: Credentialの紐付け

以下のノードでCredentialを設定：

- **Get Repository Tree**: `GitHub PAT (Obsidian)`
- **Get File Content**: `GitHub PAT (Obsidian)`
- **Get Current File SHA**: `GitHub PAT (API repo)`
- **Update GitHub File**: `GitHub PAT (API repo)`

### Step 4: テスト実行

1. ワークフローを開く
2. 右上の「Execute Workflow」をクリック
3. 初回は時間がかかる（219ファイル処理）
4. 成功すると `index.json` が更新される

---

## 📊 ワークフロー構成

```
Schedule Trigger (毎日2:00)
  ↓
Get Repository Tree (Obsidianリポジトリ全体取得)
  ↓
Filter Target Files (03_思考の森, 06_知識の海のみ抽出)
  ↓
Get File Content (ファイル内容取得)
  ↓
Extract Metadata (メタ情報抽出・全件ループ処理)
  ↓
Generate index.json (JSON生成・Base64エンコード)
  ↓
Get Current File SHA (既存ファイルのSHA取得)
  ↓
Update GitHub File (index.json更新)
```

---

## � 重要な設定ポイント
### Extract Metadata ノード（全件ループ処理）

**重要**: `$input.all()`を使って全ファイルをループ処理すること：

```javascript
const results = [];

for (const item of $input.all()) {
  const inputData = item.json;
  // ... 処理 ...
  results.push({
    path: ...,
    has_meta: ...,
    char_count: ...,
    word_hint: ...,
    tags: ...
  });
}

return results.map(r => ({ json: r }));
```

**理由**: 1件ずつ処理する`$input.item.json`では、複数ファイルを正しく処理できません。219件を正しく処理するには`$input.all()`で全件をループ処理する必要があります。
### Generate index.json ノード

JavaScriptコードの最後で**Base64エンコード済みの値**を返すこと：

```javascript
const jsonString = JSON.stringify(index, null, 2);
const base64Content = Buffer.from(jsonString).toString('base64');

return {
  json: {
    content: base64Content
  }
};
```

### Update GitHub File ノード - Body Parameters

1. **message**: `Update index.json for 雷アプリ`
2. **content**: `{{ $('Generate index.json').first().json.content }}`
3. **sha**: `{{ $('Get Current File SHA').first().json.sha }}`

**注意**: `{{` と `}}` の間にスペースなし、`=` を先頭につけない

---

## �🔍 トラブルシューティング

### エラー: "Bad credentials"
- GitHub PATが正しく設定されていない
- トークンの有効期限が切れている
- Credential IDが正しく紐付けられていない

### エラー: "API rate limit exceeded"
- GitHub APIの制限（5000 requests/hour）に到達
- 認証なしの場合は60 requests/hour
- Credentialが正しく設定されているか確認

### エラー: "File not found" (404)
- リポジトリ名が間違っている
- ブランチ名が `main` ではない
- PATの権限が不足している

### ワークフローが途中で止まる・1件しか処理されない
- Extract Metadataで`$input.all()`ループ処理を使用しているか確認
- Extract Metadataのコードで`$input.item.json`を使っていると1件しか処理されない
- Split In Batchesノードは不要（削除済み）

---

## 🎛️ カスタマイズ

### 実行タイミングの変更

Schedule Triggerノードの cron式を変更：

- 毎日2:00: `0 2 * * *`
- 毎時: `0 * * * *`
- 週1回（月曜2:00）: `0 2 * * 1`

### 対象フォルダの追加

Filter Target Filesノードの `targetPaths` を編集：

```javascript
const targetPaths = [
  '03_思考の森/',
  '06_知識の海/',
  '01_Inbox/'  // 追加
];
```

### 処理ファイル数の確認

現在の構成（直線処理）:
- **219ファイル**: 03_思考の森/ (144件) + 06_知識の海/ (75件)
- **処理結果**: Extract Metadataの`$input.all()`ループ処理により、全件を一度に処理
- **成功実績**: 2026-02-07に219件処理完了

**注意**: Split In Batchesノードは不要です（削除済み）。Extract Metadataで全件をループ処理します。

---

## ✅ セットアップチェックリスト
- [ ] n8nにCredential 2つ追加済み
- [ ] ワークフローインポート済み
- [ ] Credential紐付け完了
- [ ] テスト実行成功
- [ ] index.jsonが生成されている

---

## 🔗 生成されたAPIの確認

```bash
curl https://raw.githubusercontent.com/nayabashi/obsidian-lightning-api/main/index.json | jq .
```

成功すると：
```json
{
  "generated_at": "2026-02-07T...",
  "source": "nayabashi/Obsidian",
  "total_notes": 219,
  "notes": [...]
}
```

---

## 📝 次のステップ

1. ✅ ワークフローを定期実行に設定
2. ⏭️ lovableでWebアプリ開発
3. ⏭️ 雷プロンプト実装

