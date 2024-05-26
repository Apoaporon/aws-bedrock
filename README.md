# aws-bedrock

## Amazon Bedrockの機能
<img src=https://assets.st-note.com/img/1708999128925-eRd62bfyom.png>
- openAIのモデルは利用できない
- RAG, Agentの実装が可能
## RAGの実装
<img src=https://assets.st-note.com/img/1709003490736-646eypam9T.png>

[参考リンク](https://note.com/roku385/n/n80e16337a400)

### 記述スクリプトのフロー[参考サイト](https://zenn.dev/spiralai/articles/a189f4cbe6541c)
1. 1つのクエリ（入力プロンプト）から複数の類似クエリを作成する
2. 以下のような辞書を作成
    - {検索クエリ：ナレッジ検索結果}
3. 最終的なクエリの作成
4. 解答を得る

## 介入できそうなところ
- データの前処理
    - チャンキングの仕方？現状はデフォルトのチャンキングが実施されているので、関係ない文書がつながっている可能性あり。適切な分割ができれば、ナレッジが効果的に検索できる可能性大？
- 実装面
    - 投入promptの工夫？プロンプトエンジニアリング
    - RAG Fusion自体の手法を変えてみる？現状はただ類似クエリを生成して、検索を実施している手法を採用
- 初期クエリをもう少し具体的なものにすることで、生成される類似クエリも具体的なものになる可能性あり
- その他
    - 実装自体がRAGなので、一般的な実装戦略は一通り行うべき?
    - [参考リンク](https://qiita.com/jw-automation/items/045917be7b558509fdf2)
    - [参考リンク2](https://zenn.dev/knowledgesense/articles/47de9ead8029ba):RAGテクニック集
    - [参考リンク3](https://zenn.dev/aidemy/articles/97d5fb6ac03a4f):langchainでのRAG fusion手法と精度

- 
## データの前処理
- ドキュメントを管理しやすいチャンクに分割するのが重要
    - チャンキング戦略
        - [参考リンク1](https://fintan.jp/page/10301/)：チャンクわけには様々な手法があることが記述されている
        - RAGの文脈におけるチャンキングとは、長いテキストをより小さなセグメント（チャンク）に分割するプロセスのことを指します。RAGでは、基本的には元のテキストではなく、分割して得られたチャンクを検索対象とします。
        - [参考リンク2](https://zenn.dev/chenchang/articles/7556d1e3129ec0):チャンク分割の手法がいくつか記載されている
        - チャンクのサイズはテキストの種類にもよるのでここは実験的に行っていくしかない。
        - awsサービスのkendraを使ってちゃんキングだけできるのか？そうじゃないならlangchainとか使うことになるのかな？
    - ナレッジベース作成時
        - 最初にナレッジベースを作成するときにどのような手法でチャンキングを行うかは画面上で設定することが可能
        - [参考リンク3](https://note.com/yutaito_opst/n/n439eb2b35f5b#cac94253-5b99-471d-8a27-3c3fda05ed06):Bedrock設定周り色々
- PDFは手動である程度加工していれる？
    - PyMuPDFを利用して、pdf -> 任意の形式のデータを作成して入れる
    - 個人的に良いと思うのは pdf -> markdown -> チャンキング
    - [参考リンク1](https://qiita.com/jamie-lemon/items/455e14f83b4f5c81034b)

## 手法
- RAG Fusion
<img src=https://fintan.jp/wp-content/uploads/2024/01/rag_fusion.png> [引用](https://fintan.jp/page/10301/)
    - ユーザのクエリに対して、LLMを利用して複数のクエリを　生成し、それらに対して、検索を実行し、結果をマージして返す。
    - サブクエリ的な分割して検索とかもできる
- HyDE(Hypothetical Document Embeddings:仮の文書埋め込み)今回は対象外なのでざっくり概要だけ
    1. 検索クエリを与えて、GPTに仮の文書を生成させる
    2. 生成した文書を埋め込みモデルを用いてエンコード
    3. エンコードした情報から実際の文書を検索

## メモ
- awsがBedrock, azureがAI Searchなだけで内容は自分達で工夫するだけの違い。
- awsがS3にデータを保存するようにAzureもデータベースサービスと連携

- [画像内の表を解析するための手法紹介](https://note.com/qunasys/n/nf9ee9a4e5d60)

- [RAGのためになるお話し](https://speakerdeck.com/yamahiro/shi-li-deshao-jie-sururagdao-ru-shi-nozhi-jian-tojing-du-xiang-shang-nokan-suo)


## ナレッジベース作成時の注意点
- S3内保存データの名前はローマ字表記である必要あり。日本語がファイル名に含まれている場合は、エラーになります。

## PyMuPDFを入れる時の注意点
- PyMuPDFを入れると包括的にfitzも入ってくるので、fitzを入れる必要はない
    - モジュールインポート時はfitzでするが、こいつをあらためて入れる必要はない
    - 謝って入れてしまうと、依存関係がおかしくなりエラーが出てしまうので注意
    - fitzをuninstallして、PyMuPDFを再インストール、pip install --upgrade --force-reinstall pymupdf