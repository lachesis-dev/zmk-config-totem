# TOTEM ZMK Config

## 概要

TOTEM は ZMK ファームウェアを使用した分割キーボードの設定リポジトリです。デフォルト構成は左側が **Central**（ホストに直接接続）で、右側が **Peripheral**（BLE で左に接続）する構成になっています。

### 現在の構成：左Central直結（ドングルなし）

```
[右/Peripheral] <--BLE--> [左/Central] <--USB or BLE--> [PC/Mac/iPad]
```

- 左半分が **Central**（ホストへ USB または BLE で直接接続）
- 右半分が **Peripheral**（左に BLE で接続）
- BLE プロファイル切り替えで最大 5 台のデバイスに接続可能

---

## ディレクトリ構成

```
zmk-config-totem/
├── .github/
│   └── workflows/                        # GitHub Actions CI/CD
├── config/
│   ├── boards/shields/
│   │   ├── totem/                        # メイン shield 定義
│   │   │   ├── Kconfig.defconfig         # Zephyr ビルド設定
│   │   │   ├── Kconfig.shield            # Shield オプション
│   │   │   ├── totem.zmk.yml             # ZMK 固有設定
│   │   │   ├── totem.dtsi                # デバイスツリー (共通)
│   │   │   ├── totem-layouts.dtsi        # キーボードレイアウト定義
│   │   │   ├── totem_left.conf           # 左側ビルド設定
│   │   │   ├── totem_left.overlay        # 左側デバイスツリー設定
│   │   │   ├── totem_right.conf          # 右側ビルド設定
│   │   │   ├── totem_right.overlay       # 右側デバイスツリー設定
│   │   │   ├── totem.conf                # 両側共通設定
│   │   │   └── totem.keymap              # キーマップ定義 (要編集)
│   │   └── totem_dongle/                 # ドングル用 shield (オプション)
│   │       ├── Kconfig.defconfig         # Zephyr ビルド設定
│   │       ├── Kconfig.shield            # Shield オプション
│   │       ├── totem_dongle.conf         # ドングルビルド設定
│   │       └── totem_dongle.overlay      # ドングルデバイスツリー設定
│   ├── .zmk.yml                          # ZMK 設定
│   ├── west.yml                          # ZMK 依存モジュール定義
│   ├── totem.conf                        # 共通設定
│   ├── totem_left.conf                   # 左側設定 (Central / USB+BLE)
│   ├── totem_right.conf                  # 右側設定 (Peripheral / トラッキング対応)
│   ├── totem_dongle.conf                 # ドングル設定 (オプション)
│   └── totem.keymap                      # キーマップ (編集用)
├── docs/
│   └── images/                           # ドキュメント画像
├── build.yaml                            # ビルドターゲット定義
└── readme.md                             # このファイル
```

---

## セットアップと使用方法

### ビルドとフラッシュ

このプロジェクトは **GitHub Actions により自動ビルド** されます。

**フラッシュ手順（左Central直結構成）**

1. GitHub Actions のビルド結果から `.uf2` アーティファクトをダウンロード
   - 左側：`totem_left` 用ファームウェア
   - 右側：`totem_right` 用ファームウェア

2. **左側をフラッシュ**（Central + USB/BLE ホスト接続）
   - 左側の Xiao にファームウェアをフラッシュ

3. **右側をフラッシュ**（Peripheral + トラッキング対応）
   - 右側の Xiao にファームウェアをフラッシュ

4. **接続確認**
   - 左側を PC に USB で接続、または BLE でペアリング
   - 右側の電源を入れる
   - 左側との自動ペアリングを確認

**ローカルでビルドする場合**

カスタマイズが必要な場合は、ZMK 開発環境でローカルビルドができます：
- 手動セットアップ：[ZMK Development Setup](https://zmk.dev/docs/development/setup) を参照

---

## 設定オプション

### BLE プロファイル切り替え

複数デバイスに同時接続できます。キーマップに以下のビヘイビアを追加：

```c
&bt BT_SEL 0   // デバイス1 (例: Windows PC)
&bt BT_SEL 1   // デバイス2 (例: Mac)
&bt BT_SEL 2   // デバイス3 (例: iPad)
&bt BT_SEL 3   // デバイス4
&bt BT_SEL 4   // デバイス5
&bt BT_CLR     // 現在のプロファイルのペアリング情報をクリア
```

### トラッキングセンサー（PMW3610）

右側にマウス機能用の トラッキングセンサー が搭載されている場合、以下の設定が有効です：

- `config/totem_right.conf` - SPI通信設定
- `config/boards/shields/totem/totem_right.overlay` - ピン配置・SPI設定

### トラッキングセンサーなしビルド

センサーを搭載していない場合は、以下をコメントアウト：

**`boards/shields/totem/totem_right.overlay`**
```
// spi設定全体
```

**`config/totem_right.conf`**
```
# CONFIG_SPI=y
# CONFIG_PMW3610=y
# CONFIG_ZMK_POINTING=y
# CONFIG_ZMK_POINTING_SMOOTH_SCROLLING=y
```

---

## ドングル構成への切り替え

ドングル（リセッター）を使用する構成に切り替えられます。

### 切り替え手順

以下のファイルを順に変更：

| ファイル | 変更内容 |
|---------|---------|
| `build.yaml` | ドングルターゲット行のコメントを外す |
| `config/boards/shields/totem_dongle/Kconfig.shield` | コメントを外す |
| `config/boards/shields/totem_dongle/Kconfig.defconfig` | コメントを外す |
| `config/boards/shields/totem_dongle/totem_dongle.overlay` | コメントを外す |
| `config/west.yml` | `zmkfirmware` の revision を `v0.3` に固定 |
| `config/totem.conf` | `CONFIG_BT_MAX_CONN` を確認・調整 |
| `config/totem_left.conf` | `CONFIG_ZMK_SPLIT_ROLE_CENTRAL=n`、`CONFIG_ZMK_USB=n` に変更 |
| `config/totem_dongle.conf` | 全行のコメントを外す |

### ドングル構成でのフラッシュ順序

```bash
# 1. ドングル (Central + USB ホスト接続)
# 2. 左側 (Peripheral + BLE)
# 3. 右側 (Peripheral + BLE)
```

---

## キーマップのカスタマイズ

キーマップは `config/boards/shields/totem/totem.keymap` で定義されています（参考ファイルは `config/totem.keymap`）。

### 編集方法

1. `config/boards/shields/totem/totem.keymap` を開く
2. レイアウトをカスタマイズ（[ZMK キーコード リファレンス](https://zmk.dev/docs/reference/keycodes)を参照）
3. ビルド・フラッシュして反映確認

### レイアウト定義

レイアウトは `totem-layouts.dtsi` で定義されています。`QWERTY`、`QWERTZ`、`AZERTY` など複数のオプションがあります。

---

## トラッキング設定（PMW3610）

トラッキングセンサー搭載版での設定：

### ハードウェア設定
- `config/boards/shields/totem/totem_right.overlay` - SPI ピン配置
- `config/totem_right.conf` - ドライバとポイント機能の有効化

### キーマップ統合
キーマップに以下を追加すると、ポインティング機能が利用できます：

```c
&mkp LCLK      // クリック
&mmv MOVE_UP   // 上移動
&mwh SCROLL_DOWN // スクロール
```

詳細は [ZMK Pointing ドキュメント](https://zmk.dev/docs/features/pointing) を参照。

---

## トラブルシューティング

### ビルドが失敗する

- **west が見つからない**: ZMK 開発環境が正しくセットアップされているか確認
- **Shield が認識されない**: `config/boards/shields/totem/Kconfig.shield` のコメントを確認
- **依存モジュールエラー**: `west update` を実行して依存関係を更新

```bash
west update
```

### フラッシュが失敗する

- **ボードが認識されない**: USB ドライバが正しくインストールされているか確認
- **ファイルが見つからない**: ビルドディレクトリに `.uf2` ファイルが生成されているか確認

### BLE が接続できない

1. 両側で `settings_reset` をフラッシュ
2. 左側を再起動して右側との再ペアリングを待つ
3. ファームウェアのバージョン不一致がないか確認

---

## 開発環境

### VS Code Dev Containers 使用（推奨）

### ローカル環境での開発

詳細は [ZMK Documentation - Development Setup](https://zmk.dev/docs/development/setup) を参照してください。

---

## 参考資料・リソース

- **オリジナルリポジトリ**: [GEIGEIGEIST/zmk-config-totem](https://github.com/GEIGEIGEIST/zmk-config-totem)
- **ドングル構成参考**: [eigatech/zmk-config (totem-dongle)](https://github.com/eigatech/zmk-config/tree/totem-dongle)
- **PMW3610 ドライバ**: [inorichi/zmk-pmw3610-driver](https://github.com/inorichi/zmk-pmw3610-driver)
- **ZMK Official Documentation**: [https://zmk.dev](https://zmk.dev)
- **ZMK ドキュメント日本語**: [https://zmk.dev/docs](https://zmk.dev/docs)

---

## 更新履歴

- **2026-06-20**: ディレクトリ構造、ビルド手順、設定オプションを実際のプロジェクト構成に更新。キーマップカスタマイズ、トラッキング設定、トラブルシューティングを追加
