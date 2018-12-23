## 4章 Goでの並行処理パターン

### 4.1 拘束

- 拘束は、情報を一つの並行プロセスからのみ得られることを確実にしてくれる
- 二種類の拘束
  - アドホック拘束
  - レキシカル拘束
- レキシカル拘束の利点
  - コストの低さ
  - コードの可読性

### 4.2 for-selectループ
- いくつかのシナリオ
  - チャネルから繰り返しの変数を送出する
  - 停止シグナルを待つ無限ループ

### 4.3 ゴルーチンリークを避ける
- ゴルーチンの終了
  - ゴルーチンが処理を完了する
  - エラーによって処理を続けられない
  - 停止するように命令される
- シグナルとしてのdoneチャネル

### 4.4 orチャネル
- doneチャネルを一つにまとめて、どれか一つが閉じられたらまとめてチャネルを閉じたい。
- 再帰とゴルーチンを使って合成したdoneチャネルを作る
- システム内で複数のモジュールを組み合わせる際のつなぎ目として利用すると良い

### 4.5 エラーハンドリング
- Goでは、エラーハンドリングが重要でプログラムを書くときはエラーの伝搬について、アルゴリズムを考えると同じくらい注意を払うべき
- 誰がそのエラーを処理する責任を持つべきか
- 取得されるであろう結果とエラーを対にする
- エラーはゴルーチンから返される値を構築する際の第一級市民として捉えられるべき

### 4.6 パイプライン
- パイプラインは特にデータストリームやバッチ処理に便利な、抽象化に使える道具
- パイプライン≒データを受け取って、何らかの処理を行い、どこかに渡す

#### 4.6.1 パイプライン構築のためのベストプラクティス
- doneチェネルの効果
  - ゴルーチンリークを避ける
  - パイプライン全体を割り込み可能にする
- ジェネレーターで個々の値をチャネル上を流れるデータのストリームに変換する
- チャネルの効果
  - 各ステージを安全に並行実行可能にする
  - どのステージでもすぐに出力を返すことを可能にする

#### 4.6.2 便利なジェネレーターをいくつか
- repeat
- take
- repeatFn

### 4.7 ファンアウト、ファンイン
- ファンアウト≒パイプラインから入力を扱うために複数のゴルーチンを起動するプロセス
- ファンイン≒複数の結果を一つのチャネルに結合するプロセス
- ファンアウト、ファンインを使うべき場合
  - そのステージがより前の計算結果に依存していない
  - 実行が長時間に及ぶ

### 4.8 or-doneチャネル
- ゴルーチンを使うことでdoneチャネルの処理をカプセル化する

### 4.9 teeチャネル
- チャネルからストリームを二つに分け、同じ値を異なる場所で使用する

### 4.10 bridgeチャネル
- チャネルのチャネルを崩して単一のチャネルにする

### 4.11 キュー
- キューはプログラムを最適化する際の最後に導入するべき技術
- キューを使うとステージがブロック状態になる時間が短くなる
- キューがシステム全体の性能を向上させうる状況
  - ステージ内でのバッチ処理によるリクエストが時間を節約する場合
  - ステージにおける遅延がシステムにフォードバックループを発生させる場合
- チャンキング
- ネガティブフィードバック、下方スパイラル、デススパイラル
- リトルの法則

### 4.12 contextパッケージ
- contextパッケージの目的
  - コールグラフの各枝をキャンセルするAPIを提供する
  - コールグラフを通じてリクエストに関するデータを渡すデータ置き場を提供する
- contextのガイドライン
  - データやプロセスはAPIの協会を通過すべき
  - データは不変であるべき
  - データは単純な方に向かっていくべき
  - データはデータであるべきでメソッド付きの型であるべきではない
  - データは修飾の操作を助けるべきものであって、それを駆動するものではない

### 4.13 まとめ