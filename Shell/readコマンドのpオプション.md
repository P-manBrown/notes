# `read`の`p`オプションはシェルによって挙動が異なる

## Bashの場合

以下のようにすることで入力を要求する際にメッセージを表示できる。  

```bash
$ read -p 'What is your project named? > ' PROJECT_NAME
What is your project named? > 
```

* Bashのread

<https://linuxcommand.org/lc3_man_pages/readh.html>

## Zshの場合

Zshで上記と同じコマンドを実行すると以下のようにエラーとなる。  

```zsh
$ read -p 'What is your project named? > ' PROJECT_NAME
zsh:read:1: -p: no coprocess
```

これは`p`オプションの挙動が異なることから生じている。  

* Zshのread

<https://zsh.sourceforge.io/Doc/Release/Shell-Builtin-Commands.html#index-read>

## 解決策

ZshでBashと同様のことを実現するには以下のように記述する。  

```zsh
$ read PROJECT_NAME'?What is your project named? > '
What is your project named? > 
```

またZshとBash両方で動作させるには以下のようにする。  

* 条件分岐

```zsh
if [ -n "$ZSH_VERSION" ]; then
  read PROJECT_NAME'?What is your project named? > '
else
  read -p 'What is your project named? > ' PROJECT_NAME
fi

echo $PROJECT_NAME
```

```zsh
if [[ $ZSH_EVAL_CONTEXT = toplevel ]]; then
  read PROJECT_NAME'?What is your project named? > '
else
  read -p 'What is your project named? > ' PROJECT_NAME
fi
```

* 別のコマンドでプロンプトを出力

```zsh
echo -n 'What is your project named? > '
read PROJECT_NAME
```

```zsh
printf 'What is your project named? > '
read PROJECT_NAME
```

## 参考

<https://stackoverflow.com/questions/15174121/how-can-i-prompt-for-yes-no-style-confirmation-in-a-zsh-script>

<https://palepoli.skr.jp/wp/2019/06/23/bashzsh-read-command/>

<https://qiita.com/isobecky74/items/63ac79a039a4975d569d>
