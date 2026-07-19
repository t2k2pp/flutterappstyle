# 姉妹アプリ共通デザイン仕様書（スタイルガイド）

対象: **pricello / breezy-recipe-stock / stocello / pacello**（以後「姉妹アプリ」）
基準: **pricello** の実装を正とする。本書の「参照実装」は pricello のファイルパスを指す。

## 本書の目的と使い方

- 姉妹アプリの UI/UX を統一し、**どのアプリでも同じ操作感**（ラーニングコスト最小）にする。
- 生成AIに機能追加・改修を依頼するときは**本書を必ずコンテキストに含める**。
  本書と異なる UI を実装してはならない。異なる UI が必要な場合は、**先に本書を更新**してから実装する
  （思い付きでの画面固有実装は禁止。アプリ固有の逸脱は各アプリの Docs に理由を残す）。
- 各アプリの Docs（例: pricello `Docs/05_design_system.md`）はアプリ内の詳細設計。
  **矛盾したら本書が優先**。

## 0. 大原則

1. **トークン参照・直値禁止**: 色は `AppColors`、文字は `AppText`、余白は `AppSpacing` を参照する。
   画面内に `Color(0xFF...)` や生の `TextStyle` を書かない。新しい色は `app_colors.dart` に追加してから使う。
2. **部品化**: `SoftCard` / `EmptyState` / `confirmDestructive` 等の共通部品を使う。画面固有の亜種を作らない。
3. **フォールバック・場当たり対応の禁止**: 失敗を握りつぶさず SnackBar で明示。ダミー値で誤魔化さない。
4. **存在しない機能を UI に出さない**: ビルド条件・プラットフォームで使えない機能は、
   無効化ではなく**カード/項目ごと非表示**にする（pricello の `AppCapabilities` / 地図ソースカード方式）。
5. **配色は目が疲れない淡いパステル基調**: 背景はほぼ白のティント、アクセントは1色、
   サムネ・タグ等の識別色は淡いパステルパレットから自動割当。

---

## 1. デザイントークン

### 1.1 カラー（役割トークン）

参照実装: `pricello/lib/core/theme/app_colors.dart`、テーマ切替は `breezy-recipe-stock/lib/core/theme/app_colors.dart`

全アプリで**同じトークン名**を定義する。配色の実値は §1.2 の共通5配色に統一し、
アプリの個性は「どれを既定にするか」で出す（**明度・彩度の設計は5配色すべてで共通**）。

| トークン | 役割 | pricello（オレンジ系）の値 | breezy グリーン系の例 |
|---|---|---|---|
| `bg` | 画面背景。ほぼ白のティント | `#FBF6F1` | `#F4FAF6` |
| `card` | カード背景 | `#FFFFFF` | `#FFFFFF` |
| `ink` | 主要テキスト（黒でなく濃い色相寄り） | `#2B2622` | `#22423F` |
| `ink2` | 二次テキスト | `#6B6059` | `#5C7370` |
| `ink3` | 三次テキスト/プレースホルダ/非選択アイコン | `#A39890` | `#98ACA8` |
| `line` | 罫線/区切り | `#EEE6DC` | `#E2EEE7` |
| `accent` | アクセント。CTA/FAB/選択状態 | `#EC6A3A` | `#2F9E8F` |
| `accentSoft` | アクセント淡色。選択背景/スプラッシュ | `#FDE9DF` | `#DFF3EE` |
| `accentDeep` | アクセント濃色。**破壊的操作ボタンの文字色** | `#C4481E` | `#1E7A6C` |
| `chipBg` | チップ非選択背景 | `#F3ECE2` | `#EAF3ED` |

- 意味が固定される semantic 色（例: 最安↓の `greenish #379B5C`、★評価、危険色）は
  役割トークンと分けて定義し、**テーマ切替でも変えない**。
- 識別用パレット（商品/タグ = 淡いパステル約9色、店舗等 = アクセント9色）を持ち、
  `palette[seed % length]` で**安定した自動割当**をする（参照: `AppColors.productColor/storeColor`）。

### 1.2 テーマ切替（全アプリ必須）

参照実装: `pricello/lib/core/theme/app_colors.dart`, `pricello/lib/providers/providers.dart`（`AppPaletteNotifier`）, `pricello/lib/app.dart`

**共通5配色**を全アプリが同じ定義・同じ並びで持つ。アプリごとに配色を発明しない。

| 配色 | id | bg | ink | ink2 | ink3 | line | accent | accentSoft | accentDeep | chipBg |
|---|---|---|---|---|---|---|---|---|---|---|
| オレンジ | `orange` | `#FBF6F1` | `#2B2622` | `#6B6059` | `#A39890` | `#EEE6DC` | `#EC6A3A` | `#FDE9DF` | `#C4481E` | `#F3ECE2` |
| グリーン | `green` | `#F4FAF6` | `#22423F` | `#5C7370` | `#98ACA8` | `#E2EEE7` | `#2F9E8F` | `#DFF3EE` | `#1E7A6C` | `#EAF3ED` |
| ブルー | `blue` | `#F3F7FB` | `#24384A` | `#5A6E80` | `#97A9B9` | `#E1EAF2` | `#3E7BC0` | `#E0ECF8` | `#2A5C99` | `#E9F0F7` |
| パープル | `purple` | `#F8F5FB` | `#3A2E4A` | `#6D6180` | `#A99BB9` | `#ECE4F3` | `#8A63C0` | `#EFE6F8` | `#6A44A0` | `#F0E9F7` |
| ピンク | `pink` | `#FDF5F8` | `#4A2A38` | `#7C5866` | `#B89AA6` | `#F3E2EA` | `#D86A93` | `#F7E3EC` | `#B84A73` | `#F6E7EE` |

（`card` は全配色 `#FFFFFF`。並びは色相環順: オレンジ→グリーン→ブルー→パープル→ピンク）

**アプリごとの既定配色**（アプリの識別色。初回起動時はこの配色が選択された状態）:

| アプリ | 既定配色 | 備考 |
|---|---|---|
| pricello | オレンジ | 従来色 |
| breezy-recipe-stock | グリーン | 従来色 |
| stocello | ブルー | 従来のティール（`#0E8C86`）は breezy と被るため移行 |
| pacello | パープル | 従来のオレンジ（`#EC6A3A`）は pricello と被るため移行 |

- ピンクはどのアプリの既定でもない共通の選択肢。既定配色の変更はアプリの識別に関わるため、先に本書を更新する。
- 6色目を足す場合はスマホ幅（約360dp）でも色玉＋ラベルが1列に収まるかを確認してから本書に追加する
  （現状5色が1列の上限。折返し自体は `Wrap` で許容）。

**実装方式**（breezy 発祥・pricello 実装済みの方式）:

- 役割トークン一式を `AppPalette`（`id`/`label` 付き immutable クラス）にまとめる。
- `AppColors` は `static AppPalette _active` を持ち、各トークンは `static Color get bg => _active.bg;` の getter にする。
- `paletteById(id)` の `orElse`（未知 id の戻し先）＝**そのアプリの既定配色**とする。
- Riverpod Notifier が `SharedPreferences`（キー `theme.palette`）へ永続化し、変更時に `AppColors.setActive` を呼ぶ。
- ルートは `KeyedSubtree(key: ValueKey(palette.id), child: RootShell())` で**配色 id をキーにサブツリーを作り直して**再描画する。
  現在タブは provider に持ち、作り直しでもタブ位置を保つ（`rootTabIndexProvider`）。
- 設定 > 表示タブの先頭に「テーマの色」カードを置き、**色選択と同じ36px色玉（選択中= ink枠3px＋白チェック。§4.2）を
  横に並べ、各色玉の下に配色名の caption** を添えて選ぶ。
  ※ SegmentedButton はセグメント幅の intrinsic 計算に icon スロットや Row 内容が反映されず、
  CJKラベル＋色玉で見切れ/オーバーフローが発生するため**使わない**（pricello実機で確認済み）。

### 1.3 タイポグラフィ

参照実装: `pricello/lib/core/theme/app_text.dart`

- フォント: **Noto Sans JP**。使用ウェイト（400/500/600/700/800）を assets に**静的同梱**し、
  `GoogleFonts.config.allowRuntimeFetching = false`。OFL ライセンスを `LicenseRegistry` に登録。
- ヘルパー `AppText` を全アプリ同名で定義:

| スタイル | 定義 | 用途 |
|---|---|---|
| `AppText.title()` | 22 / w800 / letterSpacing -0.5 | 画面タイトル |
| `AppText.sectionTitle()` | 16 / w700 | カード見出し・シート見出し |
| `AppText.body()` | 14 / w500 | 本文 |
| `AppText.caption()` | 12 / w500 / **ink3** | 補助説明・副題 |
| `AppText.style(...)` | 任意指定 | 上記で足りない場合のみ |

- 金額等の主要数値は w700、単位（円など）は 62% サイズで従属表示（参照: `PriceText`）。

### 1.4 余白・密度

参照実装: `pricello/lib/core/theme/app_spacing.dart`

- 余白は 4 の倍数。`AppSpacing`（ThemeExtension）を全アプリに置く:
  - standard: `screenPadding 16` / `cardPadding 14` / `gap 8`
  - compact: `screenPadding 10` / `cardPadding 10` / `gap 6`（`visualDensity: compact` も併用）
- 画面外周・カード内側は直値でなく `AppSpacing.of(context)` を参照。
- 設定 > 表示に「余白（標準/コンパクト）」「文字サイズ（小/中/大 = 0.9/1.0/1.15、`MediaQuery.textScaler` に乗算適用）」を置く。

### 1.5 形状・エレベーション

- カード: `SoftCard` = 白背景・角丸16・淡い影 `BoxShadow(0x08000000, blur 2, offset(0,1))`
  （参照実装: `pricello/lib/ui/widgets/soft_card.dart`。`onTap` 付きなら InkWell でラップ）。
- ボトムシート: 角丸24。FAB: 角丸18の角丸四角・`accent` 背景・白アイコン。
- ドロップダウン/検索欄: 白背景・角丸12・ボーダーレス（§3.3 の `_dense` 装飾）。

---

## 2. アプリ骨格（シェル）

参照実装: `pricello/lib/ui/screens/home_shell.dart`, `breezy .../app.dart`

- `Scaffold` + `NavigationBar`（`backgroundColor: card`、`indicatorColor: accentSoft`）で 3〜4 タブ。
  **最後のタブは必ず「設定」**。タブは outlined / filled のアイコンペア＋日本語ラベル。
- 中央コンテンツは `CenteredContent`（最大幅 640）で広幅の間延びを防ぐ。
- 画面幅 ≥840px では `NavigationRail` に切替（breezy 方式。全アプリ目標）。
- 主要アクション（撮影・追加など）はシェル共通 FAB。バッジ件数は `Badge` で表示。
- `IndexedStack` でタブ状態を保持する。

---

## 3. 一覧画面（最重要・全アプリ共通の型）

参照実装: `pricello/lib/ui/screens/home_screen.dart`

一覧画面は必ず次の**縦の並び**で構成する。省略してよいのはそのアプリに概念が無い行だけ。

```
① ヘッダー行   : タイトル＋件数キャプション | 表示切替 | 列数 | （新）リセット | 主ボタン
② 検索バー     : SoftCard型 テキスト検索
③ フィルタ行   : プルダウン（Dropdown）を横に並べる
④ ソート行     : 項目プルダウン ＋ 昇順/降順トグル
⑤ 本文        : リスト / グリッド（切替式）
```

### 3.1 ヘッダー行

- 左: `AppText.title()` のタイトル ＋ その下に `AppText.caption()` の件数（例「12件の商品 ・ 34件の記録」）。
- 右にアイコン群を並べる（この順で）:
  1. **表示切替** `IconButton`（`Icons.view_list` ⇄ `Icons.grid_view`、tooltip 付き）
  2. **列数** `PopupMenuButton<int>`（グリッド時のみ表示。選択肢: **自動 / 2列 / 3列 / 4列**。`Icons.view_column`）
  3. **条件リセット** `IconButton`（**新規・全アプリに追加**。下記 3.5）
  4. 主ボタン `FilledButton.icon`（`accent` 背景。例「＋記録」「＋追加」）

### 3.2 検索バー

- `SoftCard(padding: horizontal 14, vertical 2)` 内に `Icons.search`(18, ink3) ＋ ボーダーレス `TextField` ＋
  **入力があるときだけ** × クリアボタン（18, ink3）。
- ヒントは「◯◯名で検索」。provider と往復してもカーソルが飛ばない実装にする（`didUpdateWidget` で同期）。

### 3.3 フィルタ行（**ボタン・チップでの実装は禁止**）

- 絞り込みは `DropdownButtonFormField`（`isExpanded: true`）を `Row` + `Expanded` で横に並べる（通常2つ）。
- 未選択項目は「カテゴリ: すべて」のように**先頭に null 項目**を置く。
- 装飾は共通の dense スタイル:
  ```dart
  InputDecoration(
    isDense: true, filled: true, fillColor: AppColors.card, hintText: hint,
    contentPadding: EdgeInsets.symmetric(horizontal: 12, vertical: 8),
    border: OutlineInputBorder(borderRadius: BorderRadius.circular(12), borderSide: BorderSide.none),
  )
  ```
- 選択肢の並びは利用頻度順（例: 登録件数の多いカテゴリ順）。ラベルは `ellipsis`。

### 3.4 ソート行（**必須。実装しないことは不可**）

- 左: ソート項目の `DropdownButtonFormField`（項目は**3つ程度**に絞る。例: 更新日/名前/価格）。
- 右: `SegmentedButton<bool>`（`visualDensity: compact`、`showSelectedIcon: false`）で
  **昇順（↑）/ 降順（↓）** をトグル。項目と方向を分離し、開いたメニューが画面を覆わないようにする。

### 3.5 検索・フィルタ条件のリセット（新規追加。pricello にも追加する）

- ヘッダー行のアイコン群に `IconButton`（`Icons.filter_alt_off`、tooltip「絞り込みをリセット」）を置く。
- **検索文字・フィルタ・（ソートは既定値に）を一括で初期状態に戻す**。
- 条件が初期状態のときは非表示（またはグレーアウトではなく**非表示**を推奨。出ている＝何か絞られているサイン）。
- フィルタ notifier に `clearFilters()` を実装して呼ぶ（参照: pricello `homeFilterProvider.clearFilters`）。

### 3.6 本文（リスト / グリッド）

- **リストとグリッドの両方を必ず実装**し、①のアイコンで切替。選択状態は永続化する。
- リスト: `ListView.separated`、行間 `gap`、`padding: screenPadding.copyWith(top: 8, bottom: 96)`（FAB回避）。
  行 = `SoftCard`（サムネ | 名前(15/w700, ellipsis)＋副題(caption, 「・」結合) | 右端に主要数値＋日付/バッジ、右カラムは `maxWidth: 110`）。
- グリッド: 列数は**自動/2/3/4**。
  - 自動 = `SliverGridDelegateWithMaxCrossAxisExtent(maxCrossAxisExtent: 200)`（最小タイル幅基準。iPad で自然に増える）。
  - 固定 = `SliverGridDelegateWithFixedCrossAxisCount`。**2列固定しか無い実装は禁止**。
  - タイル高さはテキストスケール追従: `mainAxisExtent = 固定要素(110) + テキストブロック(74) × textScale`。
  - タイル = `SoftCard(padding: 10)`、中央サムネ(60) → 名前 → 副題（空でも半角スペースでレイアウト維持）→ `Spacer()` → 数値＋バッジ。
- タップで詳細画面へ `MaterialPageRoute` push。

### 3.7 空状態・初回ロード

- 初回ロード（ストリーム未エミット）は `CircularProgressIndicator`。**「まだありません」をフラッシュさせない**。
- `EmptyState`（絵文字48 ＋ message ＋ subMessage ＋ 任意の `OutlinedButton`）を共通部品で:
  - 未登録: 「まだ◯◯がありません」＋ 開始手順 ＋ **「サンプルデータを試す」アクション**。
  - 絞り込みゼロ件: 「条件に一致する◯◯がありません」＋ **「絞り込みを解除」アクション**（`clearFilters` を呼ぶ）。

---

## 4. 追加・編集・削除

### 4.1 マスタ管理画面の型（**マスタ系があるアプリは管理画面を必ず実装**）

参照実装: `pricello/lib/ui/screens/stores_screen.dart`, `categories_screen.dart`, `payment_methods_screen.dart`

- 独立画面（`Scaffold` + `AppBar(title: '◯◯管理')`）。設定 > マスタタブから遷移。
- 追加は **FAB（accent・＋）**。
- 一覧行 = `SoftCard`: 識別ビジュアル（色ドット16 or 絵文字サムネ）| 名前＋メモ(caption, ellipsis) | 状態アイコン | 削除 `IconButton(Icons.delete_outline, ink3)`。
- **行タップ = 編集**（ボトムシート）。
- 空状態は `EmptyState`（絵文字＋「◯◯がありません」）。

### 4.2 追加/編集ボトムシートの型

- `showModalBottomSheet(isScrollControlled: true)` ＋ `viewInsets.bottom` パディング（キーボード回避）。
- 構成: 見出し `sectionTitle`「◯◇を追加 / ◯◇を編集」→ 名前 `TextField`（追加時 `autofocus`）→
  色選択（パレットを `Wrap` した **36px 色ドット、選択中は ink 枠3px＋白チェック**）/ 絵文字選択 → 任意項目 →
  保存 `FilledButton`（`accent`、`minimumSize: Size.fromHeight(48)`）。
- 保存失敗は SnackBar 明示＋ガード解除で再試行可能に。二重送信は `_saving` ガード。

### 4.3 削除

- **必ず共通の `confirmDestructive`** を使う（参照実装: `pricello/lib/ui/dialogs/confirm_dialog.dart`。
  AlertDialog、キャンセル/確定の2ボタン、確定文字色 = `accentDeep`）。画面ごとの独自ダイアログ禁止。
- 参照されているデータは**確認前に判定**し、「削除できません」ダイアログ（閉じるのみ）を出す。
  確定後に失敗させない。
- 全削除を伴う操作（インポートの置換等）は**二段確認**。
- 結果は SnackBar で件数付きで報告（「◯件を削除しました」）。

### 4.4 絵文字設定（**絵文字を持つエンティティでは必須**）

参照実装: `pricello/lib/ui/widgets/emoji_picker_sheet.dart`, `product_thumb.dart`

- 共通関数 `pickEmoji(context)`: `emoji_picker_flutter` をボトムシート（高さ320）で出し、選択絵文字を返す。
- サムネ部品（`ProductThumb` 相当）の描画優先順位: **画像 →（設定次第）→ 自身の絵文字 → 親カテゴリの絵文字 → 既定絵文字**。
  絵文字は `FittedBox` で縮小、背景はパステル自動割当色、角丸 = size×0.24、微ボーダー `0x0A000000`。

### 4.5 画像対応（**画像を扱うエンティティでは以下を揃える**）

- ローカル保存（アプリ専用ディレクトリ）＋ DB にはパスのみ。起動時に**孤児画像 GC**（失敗してもアプリ継続、ログは残す）。
- 一覧サムネの「写真優先 / アイコン固定」を設定 > 表示で切替可能に。
- 「画像を縮小して保存」トグルを設定 > 記録に（既定 OFF=高解像度。適用範囲は ON 後の新規のみ、と説明文に明示）。
- サムネタップまたは詳細から**フルスクリーンビューア**（ピンチズーム）へ。
- 画像読込失敗時は絵文字ボックスへ `errorBuilder` で戻す（クラッシュ・空白禁止）。

---

## 5. 設定画面（**1枚縦長は禁止。タブ分け必須**）

参照実装: `pricello/lib/ui/screens/settings_screen.dart`

- タイトル「設定」＋ `TabBar`（`labelColor: ink` / `unselectedLabelColor: ink3` / `indicatorColor: accent`）。
- タブは **表示 / 記録 / マスタ / データ / 情報** の5つを基本形とする。
  「記録」だけはアプリのドメインに合わせて改名・省略可。他の4つは全アプリ共通。

| タブ | 内容 |
|---|---|
| 表示 | **テーマの色**（§1.2）・一覧サムネ表示・文字サイズ・余白・（機能があれば表示系設定） |
| 記録 | 入力/記録に関わる設定（OCR、既定値、画像縮小 等） |
| マスタ | 各マスタ管理画面への遷移タイル |
| データ | エクスポート/インポート・クリーンアップ・**サンプルデータ** |
| 情報 | アプリ情報カード・プライバシーポリシー・OSSライセンス・サポート窓口 |

- 設定カードの型 = `SoftCard`:
  `sectionTitle` 見出し → `caption` 説明文 → 12px → 操作部品。
  - 択一 = `SegmentedButton`（**ラジオ/ドロップダウンにしない**）
  - ON/OFF = 見出し行の右端に `Switch`（`activeThumbColor: accent`）
  - 実行系 = `OutlinedButton.icon`（破壊的なら `foregroundColor: accentDeep`）
- 遷移タイルの型: `SoftCard(onTap)` 内に `Icon(accent)` + ラベル + `Icons.chevron_right(ink3)`。
- 説明文の原則: 挙動・適用範囲・例外を利用者の言葉で書く。**ビルド条件などの開発者向け注記は書かない**。

### 5.1 サンプルデータカード（**2列ボタン。2行に積まない**）

- `caption` 説明 → `Row` に `Expanded` ×2 で **「サンプル投入」「サンプル削除」の `OutlinedButton.icon` を横並び**。
- 投入済みなら投入ボタンを無効化（二重投入防止）。削除側は `foregroundColor: accentDeep` ＋ `confirmDestructive`。
- サンプルは実データと区別して管理し、一括削除可能。実データが参照中の分は残して件数を報告。

---

## 6. フィードバック・状態管理の作法

- 失敗は `ScaffoldMessenger` の SnackBar で「◯◯に失敗しました: $e」。成功も件数付きで報告。
- 非同期操作中は `_busy` フラグでボタン無効化。`mounted` チェックを徹底。
- 破壊的・自動では走らせたくない処理（GC・一括削除）は**手動トリガー＋確認**。

## 7. オーバーフロー・堅牢性規約（全アプリ共通）

- すべての `Text` に `maxLines` + `overflow: ellipsis`（または fade）を明示。
- `Row`/`Column` 内の可変幅テキストは `Expanded`/`Flexible` で包む。
- 1行に収めたい数値は `FittedBox(scaleDown)`。可変長ラベル群は `Wrap` か横スクロール。
- グリッドタイル高さはテキストスケール追従（§3.6）。
- ウィジェットテストで「幅 × textScaler(1.0/1.3/1.6)」マトリクスの overflow 例外なしを検証
  （参照: pricello `test/widget/overflow_matrix_test.dart`）。

## 8. アクセシビリティ

- 最小タップ領域 44×44。アイコンボタンには必ず `tooltip`。
- 端末の文字拡大設定を尊重（アプリ設定は乗算）。
- accent 上の白文字などコントラストを確認。

---

## 9. 実装チェックリスト（生成AIはレビュー時にこれで自己検査する）

一覧画面:
- [ ] フィルタが**プルダウン**である（ボタン・チップではない）
- [ ] ソート（項目プルダウン＋昇順/降順トグル）がある
- [ ] リスト⇄グリッド切替があり、グリッド列数は**自動/2/3/4**から選べる
- [ ] 検索バー（SoftCard型・×クリア付き）がある
- [ ] ヘッダーに**条件リセットのアイコン**がある
- [ ] 空状態が「未登録」と「絞り込みゼロ件」で分岐し、それぞれアクション付き

設定画面:
- [ ] **タブ分け**されている（表示/記録/マスタ/データ/情報）
- [ ] テーマの色切替がある（§1.2、色玉付き SegmentedButton）
- [ ] サンプル投入/削除が**横並び2列**の OutlinedButton
- [ ] マスタ管理画面（FAB追加・行タップ編集シート・削除確認）がある

共通:
- [ ] 色/文字/余白がトークン参照（直値なし）
- [ ] 絵文字設定（pickEmoji）と画像対応（§4.4/4.5）が該当エンティティにある
- [ ] 削除は `confirmDestructive`、失敗は SnackBar
- [ ] Text に maxLines/ellipsis、可変幅は Expanded

## 10. 過去に統一されなかった実装（禁止事項として明記）

| ❌ 禁止 | ✅ 正 |
|---|---|
| 一覧フィルタをボタン/チップの列で実装 | プルダウン2つを横並び（§3.3） |
| ソート機能を実装しない | 項目プルダウン＋昇順/降順トグル（§3.4） |
| グリッド2列固定 | 自動/2/3/4列を選択可（§3.6） |
| 設定を1枚の縦長画面に実装 | 5タブ構成（§5） |
| サンプル投入/削除を縦2行のボタン | 横並び2列の OutlinedButton（§5.1） |
| マスタ管理機能を作らない | 管理画面＋編集シートの型で実装（§4.1） |
| 絵文字設定を省略 | pickEmoji＋サムネ部品（§4.4） |
| 画像対応を省略・不完全実装 | 保存/GC/縮小設定/ビューア一式（§4.5） |
| 画面固有の色直値・独自ダイアログ | トークン＋共通部品（§0） |

## 11. 本書の運用

- 新しい UI パターンが必要になったら、**実装前に本書へ追記**し、姉妹アプリ全体で使える形に一般化する。
- アプリ固有でどうしても逸脱する場合は、そのアプリの Docs に「本書◯章からの逸脱と理由」を明記する。
- pricello の実装が変わったら本書も更新する（pricello が基準実装）。
