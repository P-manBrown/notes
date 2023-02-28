# 複数の要素をglobなしで引数としてコマンドに渡す方法

## 渡す方法

配列を使用することでglobなしで複数要素をコマンドの引数として渡せる。  
globを使用するとクォートした際に展開されないが以下の方法であれば複数のファイルを同時に`mv`できる。  
また移動するファイルが1つでもエラーが発生したりしない。  

```bash
mkdir_and_mv() {
  source=($1)
  target="$2"
  if [[ ! -e "${target%/*.yml}" ]]; then
    mkdir -pv "${target%/*.yml}" \
      | grep -m 1 -Eo "'.+'" \
      | tr -d "'" \
      | xargs chown -R "${USER_NAME}"
  fi
  mv -n "${source[@]}" "${target}"
}

cache_paths="$(find "${HOME}/.solargraph/cache" -mindepth 1 -maxdepth 1)"
mkdir_and_mv "${cache_paths}" "${SOLARGRAPH_CACHE}"

```
