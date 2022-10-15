# Railsにおける`--mount=type=cache`の使い方

Railsにおいて`$GEM_HOME`に`--mount=type=cache`を設定するには以下のようにする。

```Dockerfile
...

WORKDIR /home/${USER_NAME}/${PROJECT_NAME}
COPY --chown=${USER_NAME} Gemfile* /home/${USER_NAME}/${PROJECT_NAME}/
RUN --mount=type=secret,uid=1000,id=github-pkg-cred,required \
    --mount=type=cache,uid=1000,target=/home/${USER_NAME}/.cache/bundle <<-EOF
  set -eu
  gem update --system ${RUBYGEMS_VERSION}
  export BUNDLE_RUBYGEMS__PKG__GITHUB__COM=$(cat /run/secrets/github-pkg-cred)
  GEM_HOME=/home/${USER_NAME}/.cache/bundle bundle install
  cp -aRT /home/${USER_NAME}/.cache/bundle ${GEM_HOME}
EOF

...

```

一度`~/.cache/bundle`にインストールしそれを`$GEM_HOME`にコピーする。

* `GEM_HOME=/home/${USER_NAME}/.cache/bundle bundle install`  
  `GEM_HOME`を変更しないとインストールされるパスが`bundle/ruby/3.1.0/` となる。  
  `ruby/3.1.0/`があると`GEM_HOME`へにコピーする際にディレクト構造が異なりうまくいかない。  
  そのため一時的に`GEM_HOME`を変更しディレクトリ構造が一致するようにしている。  

* `cp -aRT /home/${USER_NAME}/.cache/bundle ${GEM_HOME}`  
  `-a`はサブディレクトリや属性なども含め、可能な限りすべてを保持しながらコピーする  
  `-R`はコピー元にディレクトリを指定した場合、再帰的に（サブディレクトリも含めて）コピーする  
  `-T`はコピー先（最後の引数）がディレクトリでも特別扱いしない  
