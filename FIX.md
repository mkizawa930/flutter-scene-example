# ビルド成功までの修正一覧

## 1. FVMセットアップ

- `fvm install master` → `fvm use master`
- Flutter masterチャンネルが必要（`flutter_scene`がFlutter GPUを使用するため）

## 2. pubspec.yaml — ローカルパス依存をpub.devに変更

```diff
- flutter_scene:
-   path: ../flutter_scene
- flutter_scene_importer:
-   path: ../flutter_scene/importer
+ flutter_scene: ^0.9.2-0
+ flutter_scene_importer: ^0.9.0-0
```

## 3. pubspec.yaml — 廃止パッケージ削除

```diff
- native_assets_cli: ^0.8.0
```

`native_assets_cli`は`flutter_scene_importer`の推移的依存として自動解決される。

## 4. インポートパス修正（7ファイル）

flutter_scene 0.9.xでは個別エクスポートが `src/` 配下に移動。全て統一バレルインポートに変更：

```diff
- import 'package:flutter_scene/node.dart';
- import 'package:flutter_scene/camera.dart';
- import 'package:flutter_scene/animation.dart';
- import 'package:flutter_scene/material/physically_based_material.dart';
- import 'package:flutter_scene/material/unlit_material.dart';
+ import 'package:flutter_scene/scene.dart';
```

対象ファイル:

- `lib/demo/game.dart`
- `lib/demo/player.dart`
- `lib/demo/camera.dart`
- `lib/demo/coin.dart`
- `lib/demo/spike.dart`
- `lib/demo/spawn.dart`
- `lib/demo/resource_cache.dart`

## 5. hook/build.dart — native_assets_cli API変更対応

```diff
- build(args, (config, output) async {
-   buildModels(buildConfig: config, inputFilePaths: [
+ build(args, (input, output) async {
+   buildModels(buildInput: input, inputFilePaths: [
```

## 6. flutter_scene_importer パッチ（pub-cache内）

`~/.pub-cache/hosted/pub.dev/flutter_scene_importer-0.9.0-0/lib/build_hooks.dart` から廃止された実験フラグを削除：

```diff
-   '--enable-experiment=native-assets',
    'run',
```

> **注意**: pub-cacheへの直接パッチなので、`flutter pub cache clean` 等で消える。`flutter_scene_importer`の次バージョンで修正されるべき問題。

## 7. Info.plist — Flutter GPU有効化

### macOS (`macos/Runner/Info.plist`)

```xml
<key>FLTEnableFlutterGPU</key>
<true/>
```

### iOS (`ios/Runner/Info.plist`)

```xml
<key>FLTEnableImpeller</key>
<true/>
<key>FLTEnableFlutterGPU</key>
<true/>
```

## 8. ios/Podfile — デプロイターゲット変更

```diff
- # platform :ios, '13.0'
+ platform :ios, '16.0'
```

## 既知の問題

- **iOSシミュレーター**: `flutter_soloud`パッケージの`-G`コンパイラフラグがシミュレーターで非対応。実機では問題なし。
- **レイアウトオーバーフロー**: ウィンドウサイズが小さいと `game.dart:618` の `Row` がオーバーフローする。
