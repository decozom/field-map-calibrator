# Field Timer / Field Calibrator 運用手順まとめ

このリポジトリは以下の2つで構成されている。

- **Field Calibrator**
  - フィールド画像に対して GPS 基準点（anchors）を設定するツール
- **Field Timer**
  - キャリブレーション済み JSON を読み込み、GPS 位置をマップ上に表示するタイマーアプリ

---

## 全体の考え方（重要）

- **マップ画像の位置・サイズ・基準は Calibrator と Timer で完全に共通**
- キャリブレーターは「設定を作るだけ」
- タイマーは「JSONを読むだけ」
- Timer 側にフィールド固有の値を直書きしない

---

## ディレクトリ構成（想定）

/field-timer
/fields
fields.json        ← フィールド一覧
oasis.json         ← 各フィールド定義
/assets
oasis.jpg          ← マップ画像

/field-calibrator
/maps
oasis.jpg          ← キャリブ用画像
manifest.json      ← 画像一覧

---

## キャリブレーターでやること

1. **maps フォルダにマップ画像を置く**
2. manifest.json に画像名を追加
3. キャリブレーターを開く
4. マップ画像を選択
5. P0〜P3 を実際の地形上の同一点に合わせる
6. 各ピンに対応する **lat / lon** を入力
7. 画像倍率を調整（見やすい倍率に）
8. 以下を含む JSON を生成する

---

## キャリブレーター出力 JSON 形式

```json
{
  "id": "oasis",
  "name": "oasis",
  "image": "assets/oasis.jpg",
  "defaultImgScale": 0.35,
  "anchors": [
    { "label": "P0", "x": 65.0,  "y": 74.0,  "lat": 35.865547, "lon": 139.473143 },
    { "label": "P1", "x": 279.0, "y": 35.0,  "lat": 35.865833, "lon": 139.473529 },
    { "label": "P2", "x": 266.0, "y": 285.0, "lat": 35.865175, "lon": 139.474076 },
    { "label": "P3", "x": 90.0,  "y": 311.0, "lat": 35.864925, "lon": 139.473733 }
  ]
}

・各項目の意味

id:                フィールド識別子
name:               表示名
image:              Field Timer 側で読み込む画像パス
defaultImgScale:    Timer 初期表示倍率（省略時は 1.0）
anchors:            GPS ↔ マップ変換用の4点

Field Timer 側でやること
	1.	キャリブで作った JSON を /fields に配置
	2.	fields/fields.json にエントリ追加
    
    [
      { "name": "oasis", "file": "oasis.json" }
    ]

	3.	マップ画像を /assets に配置
	4.	Field Timer を起動
	5.	フィールド選択 → 自動反映

defaultImgScale について（重要）
	•	Field Timer の JS に直書きしない
	•	必ずマップ JSON に記述する
	•	Timer 側は以下のロジックで自動処理する

     document.documentElement.style.setProperty(
     "--imgScale",
     String(data.defaultImgScale ?? 1.0)
  );

挙動
jsonに値あり:  指定倍率で表示
jsonに値なし:  1倍で表示

再キャリブレーション手順
	1.	Calibrator で再調整
	2.	JSON を上書き
	3.	Field Timer はコード変更不要

⸻

注意事項
	•	anchors は 必ず4点
	•	ピンは同一地形上の点を選ぶ
	•	defaultImgScale は「見た目用」であり計算には影響しない

⸻

最終思想（設計メモ）
	•	キャリブレーター = 設定生成器
	•	Field Timer = 設定再生器
	•	ロジックとデータを混ぜない