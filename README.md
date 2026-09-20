# cjong4-workspace

cjong4と関連ライブラリを横並びで管理する開発用ワークスペースです。
各プロジェクトは独立したGitリポジトリとして維持し、このリポジトリではサブモジュールの参照コミットで組み合わせを固定します。

## プロジェクト

| ディレクトリ | 役割 |
| --- | --- |
| [cjong4](cjong4/) | 4人打ちリーチ麻雀のコア。ルール、状態遷移、合法手生成、得点計算、対局進行 |
| [cjong4-opponent](cjong4-opponent/) | コアのmanager APIを使う対局プレイヤー群 |
| [cjong4-mjai](cjong4-mjai/) | MJAIイベントの表現・エンコード・デコード |
| [cjong4-move-evaluator](cjong4-move-evaluator/) | 合法手の価値評価、自己対局データ生成、学習・推論 |
| [cjong4-discard-risk](cjong4-discard-risk/) | 公開情報に基づく打牌の放銃危険度評価、学習・推論 |
| [cjong4-web](cjong4-web/) | WebAssemblyによるブラウザ対局・視覚的デバッグ |

## セットアップ

Gitを用意し、サブモジュールを含めて取得します。

```sh
git clone --recurse-submodules https://github.com/sh-lab/cjong4-workspace.git
cd cjong4-workspace
```

すでにcloneしている場合は、ワークスペースのルートで実行します。

```sh
git submodule update --init --recursive
```

`cjong4-mjai`・`cjong4-move-evaluator`・`cjong4-discard-risk` は、実装がある `dev/initial` ブランチのコミットを登録しています。
通常の `git submodule update` はブランチの最新ではなく、親に記録されたコミットをチェックアウトします。

## ビルド・テスト

現在はリポジトリの集約とバージョンの固定を行います。ルートの一括ビルド・統合テスト設定はまだありません。
必要なツールとビルド・テスト手順は、各プロジェクトのREADMEを参照してください。
コアのビルド・テスト例は次のとおりです（CMake 3.16以降とC11対応コンパイラが必要です）。

```sh
cmake -S cjong4 -B cjong4/build -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTS=ON
cmake --build cjong4/build --config Release --parallel
ctest --test-dir cjong4/build -C Release --output-on-failure
```

`cjong4-opponent` と `cjong4-web` は、横並びの依存ソースを使用します。
各プロジェクト内部のサブモジュールは廃止し、参照コミットはこの親リポジトリで管理します。
そのほかのライブラリを単体でビルドする場合の依存パス指定は、各READMEを参照してください。

## 更新と開発

親リポジトリに記録された組み合わせへ更新するには、作業中の変更をコミットまたは退避してから実行します。

```sh
git pull --ff-only
git submodule update --init --recursive
```

サブモジュールは通常detached HEADになります。変更するときは、対象のリポジトリ内で作業ブランチを作成します。
たとえばコアを変更する場合は次のようにします。

```sh
git -C cjong4 switch -c feature/my-change
```

変更は対象リポジトリでコミットし、そのコミットをリモートへpushした後、親で参照を更新します。
以下は子のコミット・pushが済んだ後に、ワークスペースのルートで実行する例です。

```sh
git add cjong4
git commit -m "Update cjong4 revision"
git push
```

これにより、ほかの開発者も同じコミットの組み合わせを取得できます。
`git submodule update --remote` は参照先を最新へ進める操作なので、意図的に依存を更新するときに使用し、変更内容と動作を確認してから親に記録します。

## ライセンス

この親リポジトリのファイルは [MIT License](LICENSE) で提供します。
各サブモジュールおよび同梱する第三者のコード・データには、それぞれのライセンスが適用されます。
