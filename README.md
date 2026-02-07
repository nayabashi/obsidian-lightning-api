# ⚡️ Obsidian Lightning API

Obsidian「思考の森」から雷を落とすためのメタデータAPI。

## 📌 概要

このリポジトリは、プライベートなObsidian Vaultから抽出した**意味づけを最小化したメタ情報**を提供します。

- 整理・検索・効率化は目的としない
- 意味は生成されうるが、固定されない
- 雷は自然現象であり、成果を保証しない

## 📂 データ構造

### `index.json`

```json
{
  "generated_at": "2026-02-07T12:00:00Z",
  "source": "nayabashi/Obsidian",
  "total_notes": 219,
  "notes": [
    {
      "path": "03_思考の森/example.md",
      "has_meta": true,
      "char_count": 1234,
      "word_hint": "冒頭100文字のヒント...",
      "tags": ["AI", "思考"]
    }
  ]
}
```

## 🔄 更新頻度

- **自動更新**: 1日1回（n8nワークフローによる定期実行）
- **手動更新**: 必要に応じてトリガー可能

## 🚀 利用方法

### 直接取得

```bash
curl https://raw.githubusercontent.com/nayabashi/obsidian-lightning-api/main/index.json
```

### JavaScriptから取得

```javascript
const response = await fetch('https://raw.githubusercontent.com/nayabashi/obsidian-lightning-api/main/index.json');
const data = await response.json();
console.log(data.notes);
```

## 🔐 プライバシー

- **本文は含まない**: 冒頭100文字のヒントのみ
- **メタ情報のみ**: ファイルパス、文字数、タグ等
- **元リポジトリはプライベート**: `nayabashi/Obsidian`

## 🌩️ 雷アプリとの連携

このAPIは[⚡️雷アプリ](lovable URL予定)で使用されます。

- 距離のある思考を衝突させる
- 意味づけを最小化したまま仮説を生成
- 整理・効率化は行わない

## 📄 ライセンス

MIT License

---

*このAPIは「道具」ではなく、「天候」である*
