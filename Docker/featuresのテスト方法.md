# DevContainersのFeaturesをテストする方法

## テスト方法

たとえば以下のようにtestディレクトリを作成する。  
`test`ディレクトリ内のディレクトリ名はテストするFeatureと対応していなければならない。  

```terminal
$ tree
.
├── src
│   ├── git
│   │   ├── devcontainer-feature.json
│   │   └── install.sh
...
├── test
│   ├── _global
│   │ ├── scenarios.json
│   │ └── some_test_scenario.sh
│   ├── git
│   │   ├── scenarios.json
│   │   ├── install_dotnet_and_oryx.sh
│   │   └── test.sh 
...
```

`test/test.sh`を以下のように作成する。  

```bash
#!/bin/bash

set -e

source dev-container-features-test-lib

check "version" git --version

reportResults
```

- `check <LABEL> <cmd> [arg...]...`  
`<cmd>`を実行し終了コードに応じて成功/失敗を表示する。

- `reportResults`
`check`と`checkMultiple`の結果を出力する。

テストを実行するにはDevContainersCLIを使用する。  

```terminal
devcontainer features test --features Feature名 --base-image ベースイメージ .
```

```terminal
devcontainer features test --features git --base-image mcr.microsoft.com/devcontainers/base:debian .
```

上記の場合には`git`Featureをdebianのベースイメージを使用してテストする。  

以下のようにコマンドを短くできる。  

```terminal
devcontainer features test -f git -b debian
```

## シナリオ

オプションを指定してテストを実行するには`scenarios.json`とそれに対応するファイルを作成する。  

- `scenarios.json`

```json
{
  "git-scenario": {
      "image": "ubuntu:focal",
      "features": {
          "git": {
            "version": "2.20.1"
          }
      }
  }
}
```

- `git-scenario.sh`

```bash
#!/bin/bash

set -e

source dev-container-features-test-lib

check "version" git --version

reportResults
```

`scenarios.json`の`"git-scenario"`には`.sh`のファイル名を記述する。`"git"`にはFeature名を記述する。  

例では`git`Featureを`git-scenario.sh`の内容でテストすることになる。  

なお`scenarios.json`に対応するファイルが存在しない場合にはテスト実行時に以下のようなエラーが出る。  

```terminal
$ devcontainer features test --features git-from-src-fast --base-image mcr.microsoft.com/devcontainers/base:debian .

...

[-] No scenario test script found at path '/workspaces/devcontainers-features/test/git-from-src-fast/git-from-src-fast.sh'.
  Either add a script to the test folder, or remove from scenarios.json.
```

`scenarios.json`は作成しなくても良い。  

## 参考

<https://github.com/devcontainers/cli/blob/main/docs/features/test.md>
