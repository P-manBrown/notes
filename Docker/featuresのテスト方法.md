# DevContainersのFeaturesをテストする方法

以下のようにtestディレクトリを作成する。  

```sh
$ tree
.
│
├── test
│   ├── git-from-src-fast
│   │   ├── git-from-src-fast.sh
│   │   ├── scenarios.json
│   │   └── test.sh
│   └── _global
│       └── color_and_hello.sh

```

`test/git-from-src-fast`内ファイルの内容は以下のとおり。  

- git-from-src-fast.sh

```zsh
#!/bin/bash

set -e

# Optional: Import test library
source dev-container-features-test-lib

check "version" git --version
check "gettext" dpkg-query -l gettext

# Report result
reportResults
```

- scenarios.json

```json
{
  "git-from-src-fast": {
      "image": "ubuntu:focal",
      "features": {
          "git": {
            "version": "latest"
          }
      }
  }
}

```

上記の`"git-from-src-fast"`には`.sh`のファイル名を記述する。  

ファイルが存在しない場合にはテスト実行時に以下のようなエラーが出る。  

```zsh
$ devcontainer features test --features git-from-src-fast --base-image mcr.microsoft.com/devcontainers/base:debian .

...

[-] No scenario test script found at path '/workspaces/devcontainers-features/test/git-from-src-fast/git-from-src-fast.sh'.
  Either add a script to the test folder, or remove from scenarios.json.
```

- test.sh

```zsh
#!/bin/bash

set -e

# Optional: Import test library
source dev-container-features-test-lib

# Definition specific tests
check "version" git --version

# Report result
reportResults

```

`scenarios.json`は作成しなくても良い。  
