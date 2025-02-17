# Rocket.Chatのアップデート (6.6.0 → 6.9.2)

2024/07/22  
名前：山田純平

### 1. 概要  
本資料は，Rocket.Chatのアップデート手順について述べる．元バージョンは6.6.0であり，新バージョンは6.9.2である．  
.envを変更し，コンテナを起動した．結論として，6.9.2にアップデートし，動作することを確認した．

### 2. アップデートする際の必要情報と注意事項  
　本章は，Rocket.Chatをアップデートするにあたり必要となる情報について述べる．先ず，Rocket.Chatに関する公式サイトについて示す．次に，Rocket.Chatをアップデートする際の注意点について述べる．

#### 2.1 Rocket.Chatに関する公式サイト  
(1) docker hub [1]  
現在，公式からリリースされているdocker imageの一覧．  
数字バージョンのみ確認すれば良い．  
(2) バージョンに対応するデータベース [2]  
Rocket.Chatの各バージョンに対する，データベース要件が記載されている．  
rainサーバでは，mongoDBを採用しているため，このバージョン項目を参照する．  
(3) Rocket.Chat_Docs [3]  
Rocket.Chatに関する公式ドキュメント．詳細な仕様まで明記されている．

#### 2.2 Rocket.Chatをアップデートする際の注意点  
Rocket.ChatのDocsで，アップデートする際の注意点について述べられている[4]．  
- メジャーバージョンをスキップしない  
- 2つ以上のマイナーバージョンをスキップしないことが理想  

なお，この注意点は，他サービスのアップデート時にも言えることである．  
大幅にバージョンを飛び越えてアップデートを実施すると，環境の違いからバグが発生するリスクが高まるため，細かくアップデートを実施する必要がある．従って，今回は，6.6.0 → 6.8.0 → 6.9.2 の流れでアップデートを実施した．

### 3. compose.ymlで利用される環境変数の設定方法  
　本章は，compose.ymlで利用される環境変数の設定方法について述べる．Rocket.Chatは，compose.yml 内の設定値を環境変数で埋め込むようにしてある．そして，環境変数の値は.envファイルで定義している[5]．アップデート先のバージョンを指定する際は.envファイルで定義している環境変数の値を書き換える．

### 4. Rocket.Chatのアップデート手順  
　本章は，Rocket.Chatのアップデートの手順について述べる．  
(1) アップデート先のバージョン確認  
　docker hub[1]を参照し，アップデート先のバージョンを選定する．  
(2) データベースが対応しているかを確認  
　Rocket.ChatのGitHub[2]を参照し，アップデート先のバージョンに，現在利用中のデータベースのバージョンが対応しているかを確認する．対応していなければ，まず，データベースをアップグレードする必要がある．今回は，mongoDB5.0（bitnami）が対応していたため，アップデートする必要がなかった．  
(3) コンテナを停止  
　compose.ymlがあるディレクトリにて，以下のコマンドを実行する  
```
docker compose down
```
(4) .envを書き換える
　.envファイルの"RELEASE"項目に記載するバージョンを変更する(必須)．今回は6.9.2に変更した．
```
RELEASE=6.9.2
```
(5) コンテナを起動
　compose.ymlがあるディレクトリにて，以下のコマンドを実行する．コマンド実行時に，docker imageはダウンロードされるため，docker pullする必要はない．
```
docker compose up -d
```

### 5. 結論
　Rocket.Chat6.6.0から6.9.2へのアップデートが目的である．そのために，docker-compose.ymlを変更し，デプロイした．
結果として，正常に動作することを確認した．

### 6. 参考文献
[1] docker hub，＜https://hub.docker.com/r/rocketchat/rocket.chat/tags＞  
[2] Rocket.Chat_GitHub，＜https://github.com/RocketChat/Rocket.Chat/releases＞  
[3] Rocket.Chat_docs，＜https://docs.rocket.chat/＞  
[4] Updating Rocket.Chat，＜https://docs.rocket.chat/deploy/deploy-rocket.chat/updating-rocket.chat＞  
[5] Compose 内の環境変数，＜https://docs.docker.jp/compose/environment-variables.html＞
