# uvについて

- [公式ドキュメント](https://docs.astral.sh/uv/)
- [uv(github)](https://github.com/astral-sh/uv)

## 概要

Pythonのパッケージマネージャーで、`uv`だけで`pip`, `pip-tools`, `pipx`, `poetry`, `pyenv`, `twine`, `virtualenv`などの代替ができる

## uvのインストールとアップデート
### uvのインストール
公式のスタンドアローンインストーラーを使用するのが簡単。macOS/Linuxは`curl`、Windowsは`PowerShell`コマンドを実行。

```
# macOS and Linux
curl -LsSf https://astral.sh/uv/install.sh | sh

# Windows
powershell -ExecutionPolicy ByPass -c "irm https://astral.sh/uv/install.ps1 | iex"
```
`pip`、`Homebrew`、`Cargo`でもインストール可能。

### uvのアップデート

```
uv self update
```

`uv --version`でバージョンの確認ができる。

## uvの基本的な使用方法

### プロジェクトの管理
uvでは`pyproject.toml`で定義されたPythonプロジェクトの管理をサポートしている。`init`で初期プロジェクトの作成が可能。

```
uv init project
```
すでにuvで作成したいプロジェクトディレクトリ下にいる場合は`uv init`で新規作成が可能。プロジェクト作成後、以下のような構成で`pyproject.toml`も自動生成される。

```
.
├── README.md
├── main.py
└── pyproject.toml
```

新たにプロジェクトを初期化する場合、親ディレクトリに`pyproject.toml`があると、新たに作成されるプロジェクトは親ディレクトリのワークスペースメンバーとして加えられる。
例えば、`pyproject.toml`があるディレクトリで、
```
uv init packages
```
<details>
<summary>log</summary>

```
Adding `packages` as member of workspace `/Users/xxx/app`
Initialized project `packages` at `/Users/xxx/app/packages`
```

</details>


とすると、以下のようなディレクトリ構成になる。
```
.
├── README.md
├── main.py
├── packages
│   ├── README.md
│   ├── main.py
│   └── pyproject.toml
└── pyproject.toml
```
カレントディレクトリの`pyproject.toml`を確認すると、
```diff
[project]
name = "app"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = []

+ [tool.uv.workspace]
+ members = ["packages"]
```
と`packages`がワークスペースのメンバーとして追加されていることが確認できる。基本的に以下のようなメリットがある

- ワークスペース内のプロジェクトが単一のロックファイルを共有するので依存関係の一貫性が保たれる
- 複数の関連プロジェクトを一元管理できるので、ビルドの効率が向上する

ワークスペースの詳細は[こちら](https://mtkn1.github.io/uv/concepts/workspaces/)

#### `uv init`のオプション
- `--app`(デフォルト): アプリケーションプロジェクトを作成する
- `--lib`: ライブラリプロジェクトを作成する
- `--buildbackend`: ビルドバックエンドの指定をする


他にも複数オプションがある。[こちら](https://docs.astral.sh/uv/reference/cli/#uv-init)を参考にするとよい。

### uvによるPythonのバージョン管理
#### Pythonのインストール
uvは複数のPythonバージョンを管理することができる。
```
uv python install 3.10 3.11 3.12
```
デフォルトで、管理されたPythonのバージョンを確認する or 最新バージョンのインストールを行う。`.python-version`ファイルが存在する場合、uvはファイルに記載されているPythonバージョンをインストールする。複数のPythonバージョンが必要な場合は`.python-versions`を定義することができる。

インストール済みのPythonを確認するには、
```
uv python list
```
で確認ができる。
```
uv python pin 3.11
```
とすることでカレントディレクトリのPythonのバージョンを指定することができる。（このとき、`.python-version`ファイルが書き換えられる）

Pythonの実行ファイルのパスを確認するときは、
```
uv python find
```
で確認できる

その他詳細は[こちら](https://mtkn1.github.io/uv/concepts/python-versions/)が参考になる

### パッケージを追加する
`uv add`でパッケージの追加が可能。このとき、`pyproject.toml`に依存関係が追加される。例えば`pandas`を追加するときは、pandasを追加したいプロジェクトディレクトリ下で以下のコマンドを叩く。
```
uv add pandas
```
`pyproject.toml`を確認するとprogectテーブルの`dependencies`にpandasが追加されていることが確認できる。
```diff
[project]
name = "project"
version = "0.1.0"
description = "Add your description here"
readme = "README.md"
requires-python = ">=3.12"
dependencies = [
+    "pandas>=2.2.3",
]
```
バージョンを指定してインストールしたい場合は、バージョン指定してインストールする。
```
uv add pandas==2.2.3
```
複数のパッケージを一度に追加することも可能で、その場合は
```
uv add numpy polars matplotlib
```
というようにすればいい。
テスト用のフレームワークやリンターなど開発用のパッケージを追加する際には`--dev`オプションをつける。
```
uv add pytest --dev
```
以下のように、`dependency-groups`に追加される。
```
[dependency-groups]
dev = [
    "pytest>=8.3.5",
]
```
既存プロジェクトだったりで、`requirements.txt`がある場合は、`-r`オプションをつける。
```
uv add -r requirements.txt
```
不要になったパッケージは`uv remove`で削除が可能。例えば`pandas`が不要になったとする。
```
uv remove pandas
```
とすると、pandasがアンインストールされ、`pyproject.toml`からも削除される。

`uv add`をすることでパッケージは即座にインストールされプロジェクト内で使用可能になる。ただ、この時点では依存関係の最適化が行われていない。そこで、`uv sync`コマンドを実行する。`pyproject.toml`の内容を基に必要なパッケージのインストールまたは更新が行われる。
```
uv sync
```
コマンド実行時に主に以下のようなことが行われる。
- プロジェクトに仮想環境がない場合は、自動的に新しい仮想環境を作成する
- 依存関係の解決が行われ、その結果を`uv.lock`に保存する

uvでは自動的に仮想環境を管理しているため、従来の`venv`などで必要であったアクティベートなどが不要。

## uvとその他のソフトウェアの統合
- [Integration guides](https://docs.astral.sh/uv/guides/integration/)
    - 日本語は[こちら](https://mtkn1.github.io/uv/guides/integration/)
### Jupyterとの統合
#### VS CodeからJupyterを使用する

まずプロジェクトを作成する。
```
uv init project
cd project
```
プロジェクトディレクトリに移動したら、`ipykernel`を開発依存関係として追加する
```
uv add --dev ipykernel
```
その後、プロジェクトをVS Codeで開く。
```
code .
```
Jupyter Notebookを作成する。おそらく、カーネルを選択するように求められるので作成した`.venv/bin/python`を選択する。これでJupyter Notebookが使用できる。

その他の場合は、[こちら](https://mtkn1.github.io/uv/guides/integration/jupyter/)を参考にするといい。

### Dockerイメージとの統合
