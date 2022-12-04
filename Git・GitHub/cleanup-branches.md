# マージ済みのローカルブランチを自動的に削除する方法

## 方法

Lefthookを使用して`lefthook.yml`に以下のように記述すると自動削除できる。  

```yml
post-merge:
  commands:
    cleanup-branches:
      run: >-
        git branch --merged | grep -Ev '^\*|main|develop' | xargs git branch -d
        && git remote prune origin
      fail_text: 'Read the error message above.'
```

## 問題点

- post-mergeのため`git rebase`でリモートを反映した場合には上記のコマンドは実行されない。  
  （`git pull --rebase`の場合には実行される。）  
- `git remote prune origin`については`git fetch --prune`や`git pull --prune`で置き換え可能。  
  また`git config --global fetch.prune true`しておくとオプションなしで同じ動作を実現可能。  
