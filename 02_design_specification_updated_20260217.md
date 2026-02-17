# HYPER ORIMA BROS 設計仕様書（更新版）

最終更新: 2026-02-18

## 1. アーキテクチャ
- 実装形態: 単一HTML（`index.html` / `super-mario.html`）
- 描画: Canvas 2D
- サウンド: Web Audio API
- コア構成:
  - `InputHandler`
  - `PhysicsEngine`
  - `TileMap`
  - `Mario` / `Enemy` / `Item`
  - `AudioEngine`
  - `GameState` + main loop

## 2. ゲーム状態
- `title`
- `intro`
- `playing`
- `clear`
- `ending`
- `gameover`

状態遷移は `update()` で分岐管理する。

## 3. 入力設計
### 3.1 共通
- `keys.left/right/jump/start...` の論理入力を保持
- `jumpPressed`（立ち上がり）、`jumpHeld`（押下継続）を毎フレーム更新

### 3.2 PC入力
- キーイベントで `left/right/jump` を更新
- 方向入力の継続フレーム数を計測し、閾値超えで `autoDash=true`

### 3.3 iPhone入力（Safari）
- `#appLayout.ios-controls` を有効化
- 下部ボタン `btnLeft/btnRight/btnJump` を `pointer` で処理
- 押下中のみ移動有効（離したら即解除）
- `setPointerCapture` と `pointerup/pointercancel/blur` で取りこぼし防止
- 同様に長押しで `autoDash=true`

## 4. レイアウト設計（iPhone）
- DOM:
  - `#gameArea`（上2/3）
  - `#controlsArea`（下1/3）
- 画面下は以下の固定配置:
  - 左: `◀`, `▶`
  - 右: 丸形 `JUMP`
- `resizeCanvas()` は `#gameArea` 実寸からスケール算出

## 5. プレイヤー設計
- プロパティ: 座標、速度、接地、向き、アニメ状態、被弾無敵、死亡状態
- 状態:
  - Small（16x16）
  - Big（16x32）
- `grow()` でBig化、`shrink()` でSmall化
- クリア時は `state.playerPower` を次ステージへ引継ぎ
- 死亡時は `state.playerPower="small"`

## 6. 物理設計
- 水平加速・減速
- 可変ジャンプ（押下時間で上昇量変化）
- 方向入力逆転時のスキッド減速
- 長押し時の自動ダッシュ速度上限を使用

## 7. ステージ設計
- `TileMap` がレベル別に地形・敵スポーン・?ブロック内容を構築
- 対象: 1-1, 1-2, 1-3, 1-4
- 1-4のみゴール判定を斧取得に変更
- 死亡時は `resetLevel()` でタイル/敵/アイテムを再生成

## 8. 敵・アイテム設計
- 敵クラス:
  - `Goomba`（スライム系）
  - `Koopa`（モグラ系）
  - `Piranha`（カタツムリ系）
  - `BossGoomba`（石ゴーレム系、32x32）
- アイテム:
  - `MushroomItem`（取得でBig）

## 9. 描画設計
- ドット絵は `spriteFixed()` でサイズ補正して管理
- パレットは `PIXEL_MAP` + `MARIO_PALETTE`
- キャラクターは添付イメージ方針の見た目へ更新

## 10. UI設計
- HUD:
  - OMARI / LIVES / WORLD / TIME
- 右上操作ガイド:
  - PC: `MOVE ARROWS`, `HOLD TO AUTO DASH`, `JUMP SPACE/X`
  - iPhone: `MOVE PAD LEFT/RIGHT`, `HOLD TO AUTO DASH`, `JUMP RED BUTTON`
- タイトル画面:
  - プレイヤーアイコン（選択カーソル）は表示しない
  - 選択行は文字色でハイライト

## 11. デプロイ設計
- Vercelプロジェクト: `hyper-orima-bros`
- 固定エイリアスURL: `https://hyper-orima-bros.vercel.app`
- 運用:
  - ローカル更新 -> `index.html` 同期 -> `vercel --prod`
  - GitHub連携時は `main` push で自動デプロイ

