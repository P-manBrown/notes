# PS4の設定方法

PS4を設定することでファイルを実行した際のログの表示を変更できる。  

## Bashの場合

```bash
export PS4='+\e[32m[\t]\e[34m[${BASH_SOURCE[0]##*/}:${LINENO}]\e[m> '
```

上記のようにすると以下のように出力される。  

```terminal
+[10:06:28][postCreateCommand.sh:43]> some command
```

## Zshの場合

```zsh
export PS4='+[%D{%H:%M:%S}]%1N:%i> '
```

```terminal
+[03:01:01]brew-update.sh:8> some command
```
