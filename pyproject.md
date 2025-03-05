# pyproject.tomlについて

## 基本用語の説明
- TOML: 設定ファイル用のフォーマット、可読性と厳密性に重点が置かれている
    - https://github.com/toml-lang/toml
    - TOMLでのテーブルはセクションのこと
- [ビルドバックエンド](https://packaging.python.org/ja/latest/glossary/#term-Build-Backend)
    - Pythonパッケージのビルドを行うライブラリ
    - ソースコードツリーを受け取って配布パッケージフォーマットに変換
        - `tar.gz`, `.whl`など

## 概要
pyproject.tomlは設定用ファイルで、Pythonプロジェクトのビルド・配布のメタデータ・設定を一括管理できる。管理には3つのテーブルが使用される。

[PEP735で`[dependency-groups]`というテーブルが追加された](https://peps.python.org/pep-0735/)

## 設定について
- `[build-system]`: ビルドバックエンドやビルドに必要な依存環境の宣言を行うテーブル
    - 強く推奨されている（以下、理由の一部）
        - [従来の方法が非推奨になった](https://packaging.python.org/ja/latest/guides/tool-recommendations/)
    - **key**
        - `requires`: ビルドシステムの実行に必要な依存関係を指定する
            - バックエンドとバージョンの指定ができる
        - `build-backend`: ビルドシステムを実行するバックエンドを指定する
        - `backend-path`: ビルドバックエンドがプロジェクト内に存在する場合にパスを指定する
    - 以下のような使用イメージ（`setuptools`を例とする）
    ```toml
    [build-system]
    requires = ["setuptools>=61.0"]
    build-backend = "setuptools.build_meta"
    ```
- `[project]`: Pythonプロジェクトのメタデータ（プロジェクト名やバージョンなど）の宣言を行うテーブル
    - **key**
        - `name`(必須): プロジェクトの名前を指定
            - プロジェクト名の比較では、大文字小文字の区別はない
            - アンダースコア・ハイフン・ピリオドは何文字続いていても同じとする
        - `version`(必須): プロジェクトのバージョンを指定
        - `description`: プロジェクトの簡単な説明を記載
        - `readme`: プロジェクトの詳細を含むREADMEのパスを指定
        - `requires-python`: プロジェクトが対応するPythonのバージョンを記載
        - `license`: プロジェクトのライセンス情報を記載
        - `authors`: プロジェクトの作者情報を指定（リスト形式）
            - 名前とメールアドレスの両方またはいずれか
        - `maintainers`: プロジェクトのメンテナー情報を指定（リスト形式）
            - 名前とメールアドレスの両方またはいずれか
        - `classifiers`: プロジェクトに合致するPyPIの分類詞（リスト形式）
            - [分類詞のリスト](https://pypi.org/classifiers/)
        - `dependencies`: プロジェクトが依存するパッケージをリスト形式で指定
        - `keywords`: プロジェクトに関連するキーワードを指定（リスト形式）
        - `dynamic`: ビルド時に動的に決定されるメタフィールドを指定（リスト形式）
    - サブテーブル
        - `[project.optional-dependencies]`: オプションの依存関係をカテゴリごとにリスト形式で指定
            - 基本機能では不要なライブラリを特定の機能や追加機能のためにオプションとして提供するイメージ。ユーザーが必要に応じて選択的にインストールすることができる
            - 以下のような場合、GUI機能を使用したいユーザーは`pip install project-name[gui]`としてインストールする
            - GUI機能を提供するために`PyQt5`が、CLIで機能を提供するために`rich`, `click`が必要とすると、以下のように宣言する
            ```toml
            [project.optional-dependencies]
            gui = ["PyQt5"]
            cli = [
            "rich",
            "click",
            ]
            ```
        - `[project.urls]`: プロジェクトに関連するURLを指定
        - `[project.scrips]`: コマンドラインで実行可能なスクリプトを定義
        - `[project.gui-scrips]`: GUIアプリケーションのエントリーポイントを定義
        - `[project.entry-points]`: プラグインや拡張機能を提供するためのエントリーポイントを指定
- `[tool]`: 各種ツール（ビルドツール、リンター、フォーマッターなど）の設定を一元管理するためのテーブル
    - 基本構造は以下
    ```toml
    [tool.tool_name]
    # ここからツール固有の設定を指定する
    ```
- `[dependency-groups]`: 
    - 従来、Pythonプロジェクトでは開発用依存関係など特定の開発環境向けの依存関係を管理する際に、`requirements.txt`ファイルや`extra`機能（`pip install project-name[gui]`の[]の設定）を使用していた。これらは以下の制約があった。
        - `requirements.txt`: 形式が標準化されていなく、ツール間での互換性に問題が生じることがあった
        - `extra`: パッケージのインストールの時のみ適用される。データサイエンスプロジェクトなどの非パッケージプロジェクトには不向き

`[project],`内の`dependencies`、`[project.optional-dependencies]`、`[dependency-groups]` の使い分けはざっくりと以下。
- `[project]内のdependencies`: プロジェクトの基本的な動作に必須の依存関係を定義する​
- `[project.optional-dependencies]`: ユーザーが特定の追加機能を利用する際に必要となるオプションの依存関係を定義する​
- `[dependency-groups]`: プロジェクトの開発、テスト、ドキュメント生成など、内部的な目的や環境に応じて依存関係をグループ化する
