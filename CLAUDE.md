# プロジェクト設定: Spring Boot + OCI デプロイ

このファイルは本プロジェクト(OCI上へのSpring Bootアプリデプロイ)専用の作業方針です。
共通の作業方針はグローバルのCLAUDE.mdを参照。個別ルールが競合する場合は本ファイルを優先する。

## 進め方(ユーザーはClaude Codeでのプログラム作成が初めて)

- ユーザーはClaude Codeを使ってのプログラム作成が初めてで、全体の進め方(何をどの順番で行うか)に不慣れ。作業の実行そのものよりも「今何が起きているか」を理解しながら進めたい段階にある。
- そのため本プロジェクトでは、通常より丁寧な進行を徹底する。
  1. **事前説明**: git操作・ファイル作成・コマンド実行など、何か作業を行う前には必ず「これから何をするか」「なぜそれが必要か」を簡潔に説明する。
  2. **確認**: 説明の後、ユーザーに「進めてよいか」を確認してから実行する。確認を飛ばしてまとめて進めない。
  3. これは通常のセッションの「大きな変更は確認してから」という基準よりも粒度を細かくしたもの — 1つ1つの作業ステップ(例: `git init`を打つ、1つのファイルを作る、1つのコマンドを実行する)ごとに区切って確認を取る。
- 専門用語が出た際に簡単な補足を添える方針(グローバルCLAUDE.md)は本プロジェクトでも引き続き適用する。
- **例外**: `CLAUDE.md`など`.md`ファイルへの進捗状況の記入・更新は、事前確認なしで行ってよい(2026-09-13にユーザーから明示指示)。実際のコード変更・コマンド実行・git操作・外部サービスへの操作(GitHubへのpushなど)は引き続き事前説明+確認を行う。
- 慣れてきて「もう確認は不要」といった指示があれば、その時点でこのセクションを更新する。

## ローカル開発ディレクトリ構成の方針(今後複数アプリを作る前提)

- 開発用ルートフォルダは `C:\dev`。今後作成するWebアプリはすべてこの配下に、1アプリ=1フォルダ=1Gitリポジトリで配置する(例: `C:\dev\hello-world`, `C:\dev\<次のアプリ名>`)。
- フォルダ名は英数字(kebab-case推奨)とし、日本語・全角文字・スペースを含めない。Maven/Gradle/Docker/Node.jsなどの開発ツールは、パスに日本語や全角文字が含まれると内部的な文字コード処理でエラーになることがあるため(実際に本プロジェクトで`mvn spring-boot:run`が日本語パスによるClassNotFoundExceptionで失敗した実例がある)。
- フォルダ名はMavenの`artifactId`(または各プロジェクトの成果物名)と揃えると、後から見て分かりやすい。
- 元々の個人的な整理分類(例: 「12_趣味」配下など)で管理したい場合は、`C:\dev\<プロジェクト名>` へのショートカットをエクスプローラーの「お気に入り」やその分類フォルダ内に作成する形で対応する(実体のパス自体は日本語を含めない)。

## 進捗状況(セッションをまたぐための記録)

Claude Codeはセッションをまたいだ記憶を持たないため、作業のキリが良いタイミングでこのセクションを更新し、
「どこまで完了したか」「次に何をするか」を常に最新の状態に保つ。別セッションで再開する際は、まずこのセクションを確認する。

- [x] CLAUDE.md作成、方針決定(インフラ・DB・進め方など)
- [x] Hello World用Mavenプロジェクトの雛形作成
  - `pom.xml`(Spring Boot 3.3.4 / Java 21 / Web + Thymeleaf)
  - `HelloWorldApplication.java`(起動クラス)
  - `HelloController.java`(`/`アクセス時に`message="Hello World"`を渡す)
  - `templates/hello.html`(Thymeleafで`message`を表示)
  - `.gitignore`
- [x] ローカル環境にJava 21 / Mavenが入っているか確認 → 未導入だったため両方インストール
  - Java 21: Amazon Corretto 21をwingetでインストール(`C:\Program Files\Amazon Corretto\jdk21.0.12_9`。既存のJava 17と共存)
  - Maven: 公式zip(3.9.16)を`C:\tools\maven`に展開して配置(Program Filesは管理者権限が必要なため回避)
  - `JAVA_HOME`をJava21のパスに、PATHにMavenのbinとJava21のbinをユーザー環境変数として設定済み
- [x] ローカルで動作確認 → `mvn clean package` → `java -jar target/hello-world-0.0.1-SNAPSHOT.jar` で起動し、`curl http://localhost:8080/` で「Hello World」のHTMLが返ることを確認。確認後プロセスは停止済み。
  - **既知の問題(解消済み)**: 当初のプロジェクトパス(`C:\01_fukuchi\...\12_趣味\...\08_ociにデプロイ`)に日本語文字が含まれており、`mvn spring-boot:run` がClassNotFoundExceptionで失敗した。`mvn clean package`でjar化してから`java -jar`で実行する方式で回避していたが、根本対応として下記の通りプロジェクトを移動した。
- [x] プロジェクトを日本語を含まないパスに移動
  - 移動先: `C:\dev\hello-world`(旧パス `C:\01_fukuchi\...\08_ociにデプロイ` から `CLAUDE.md`, `pom.xml`, `.gitignore`, `src` を移動。`target`はビルド生成物のため削除し、`mvn clean package`で再生成する)
  - 今後複数のWebアプリを作る際は `C:\dev\<プロジェクト名>` 配下にそれぞれ1フォルダ・1Gitリポジトリで作成する方針とする(詳細は下記「ローカル開発ディレクトリ構成の方針」を参照)
- [x] 移動後の動作再確認 → `C:\dev\hello-world` で `mvn clean package` → jar実行、および `mvn spring-boot:run` の両方が正常に動作することを確認(`curl http://localhost:8080/` で「Hello World」表示を確認)。**日本語パスが原因だったことが実証され、問題は完全に解消**。確認後プロセスは停止済み。
- [x] git初期化 + GitHubリポジトリ作成・連携
  - GitHub CLI(`gh`)をwingetでインストールし、ブラウザOAuthでログイン(アカウント: `yyama694`)
  - ローカルで`git init` → 初回コミット(このリポジトリのみに`user.name`/`user.email`をローカル設定。グローバルgit設定は変更していない)
  - `gh repo create hello-world --public --source=. --remote=origin --push` でGitHub上にリポジトリ作成とpushを実行
  - リポジトリURL: https://github.com/yyama694/hello-world (Public)
- [ ] OCI Compute Instance(Always Free)の作成
- [ ] VM上にJava 21実行環境・PostgreSQLをセットアップ
- [ ] jarをVMに転送し、systemdサービスとして常駐化
- [ ] パブリックIP+ポート開放設定を行い、ブラウザから外部アクセスできることを確認

## プロジェクト概要

- 目的: 学習を兼ねて、新規作成するSpring Bootアプリケーションを OCI(Oracle Cloud Infrastructure)にデプロイする。
- ソース管理: GitHub(このディレクトリでgit初期化 → GitHubリポジトリにpush)。
- デプロイは当面手動(SSH接続してjarを配置・再起動)。CI/CD自動化は将来的な検討事項とし、今は導入しない。

## 現在のマイルストーン: Hello Worldレベルのアプリ作成

- まずは大掛かりな機能を作らず、Spring Bootが手元で動くことを確認するためのHello Worldレベルのアプリを作る。
- 表示形式: ブラウザでアクセスして「Hello World」が表示されるHTML画面(Thymeleafテンプレートエンジンを使用。REST APIの素のテキスト返却ではなく、実際の画面表示に近い構成にする)。
- Mavenプロジェクトの識別子: groupId=`com.example`, artifactId=`hello-world`(仮発行。学習用途のため一般的な値を使用。後から変更可能)。
- このマイルストーンが完了したら、OCIへの実際のデプロイ(git初期化・GitHub連携・VM構築など)に進む。

## 技術スタック

- 言語/フレームワーク: Java 21 (LTS) + Spring Boot
- ビルドツール: Maven
- DB: PostgreSQL(アプリと同じVM上に自建て)
- Webサーバー: Spring Boot組み込みTomcatを直接公開(リバースプロキシなし、当面)

## インフラ構成(OCI)

- コンピュート: OCI Compute Instance(Always Free 無料枠の範囲に収める。Ampere A1(ARM)シェイプを想定 — 無料枠のOCPU/メモリが手厚いため)
  - ARM(aarch64)であることに注意。Javaは21系ならARM向けビルドで問題なく動くが、後々ネイティブライブラリを使う場合はアーキテクチャを意識する。
- OS: Compute Instanceの標準イメージ(Oracle Linux または Ubuntu。特に指定なければUbuntu LTSを想定)
- アプリ実行方式: jarを直接 `java -jar` で実行し、systemdサービスとして常駐化(再起動時の自動起動、ログはjournalctl経由)
- DB: 同一VM上にPostgreSQLをパッケージインストールして運用(別インスタンスには分離しない)
- ネットワーク公開: VMのパブリックIPに対し、OCIのセキュリティリスト/NSGでアプリ用ポート(例: 8080)を許可し、HTTPのみで公開
  - ドメイン取得・HTTPS(Let's Encrypt等)は今回は対応しない。将来必要になったらNginxリバースプロキシ導入を検討。
  - DBのポート(5432)は外部公開しない(VM内部からのみ接続可能にする)。

## デプロイ手順の方針(手動)

1. ローカルで `mvn clean package` してjarをビルド
2. `scp` 等でjarをOCI VMの所定ディレクトリに転送
3. VM上でsystemdサービスを再起動してアプリを反映
4. 動作確認は `http://<パブリックIP>:8080` へのアクセスで行う

デプロイ用のスクリプトや手順の詳細(SSH鍵の場所、VMのIP、systemdユニットファイルの内容など)は、実際に環境を構築した段階でこのファイルに追記していく。

## コーディング方針(本プロジェクト固有)

- パッケージ構成やController/Service/Repository層の分割など、Spring Bootの一般的な作法に従う。過度な抽象化はしない(グローバル方針と同様)。
- application.properties/yml にDB接続情報などの秘密情報を平文でコミットしない。ローカル用の設定と本番用の設定は分離し、パスワード等は環境変数やVM上の設定ファイル(gitignore対象)で管理する。
- .gitignore には target/, .idea/, *.iml, application-local.yml(または同等の秘匿設定ファイル)を含める。

## セキュリティ上の注意

- OCIの認証情報(APIキー、SSH秘密鍵、config)は絶対にリポジトリにコミットしない。
- VMのセキュリティリストは必要最小限のポートのみ開放する(SSH: 22、アプリ: 8080)。
- PostgreSQLのパスワードは推測されにくいものを設定し、外部公開しない。

## 未確定・今後検討する事項

- ドメイン取得・HTTPS化(Let's Encrypt + Nginx)
- CI/CD自動化(GitHub Actions等)
- DBのバックアップ方針
