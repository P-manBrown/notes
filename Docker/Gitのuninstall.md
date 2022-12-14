# Gitを最新版にする際の古いGitのアンインストールについて

Gitをアンインストールしても`apt-get install -qy gh`の際に再インストールされる。  
そのためGitをアンインストールしても特に意味はなかった。  

以下`Dockerfile`の内容。  

```Dockerfile
# syntax=docker/dockerfile:1
ARG NODE_IMAGE_TAG
FROM node:${NODE_IMAGE_TAG}

ARG GIT_VERSION=2.38.0
ARG PROJECT_NAME
ARG USER_NAME
ENV TZ=Asia/Tokyo

RUN <<-EOF
  set -ex
  rm -f /etc/apt/apt.conf.d/docker-clean
  cat <<-'EOT' > /etc/apt/apt.conf.d/keep-cache
    Binary::apt::APT::Keep-Downloaded-Packages "true";
  EOT
EOF
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
  --mount=type=cache,target=/var/lib/apt,sharing=locked <<-EOF
  set -eux
  curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
  chmod go+r /usr/share/keyrings/githubcli-archive-keyring.gpg
  echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/\
  githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
  | tee /etc/apt/sources.list.d/github-cli.list > /dev/null
  apt-get purge -y git
  apt-get autoremove -y
  apt-get update -qq
  apt-get install -qy bash-completion gettext gh libcurl4-gnutls-dev
  cd /usr/local/src
  wget https://github.com/git/git/archive/refs/tags/v${GIT_VERSION}.tar.gz
  tar -xzf v${GIT_VERSION}.tar.gz
  cd git-${GIT_VERSION}
  make prefix=/usr/local all -j "$(nproc)"
  make prefix=/usr/local install
EOF

RUN corepack enable npm

USER ${USER_NAME}
WORKDIR /home/${USER_NAME}/${PROJECT_NAME}

RUN gh config set editor "code -w"

RUN <<-EOF
 set -eux
 cat <<-'EOT' >> /home/${USER_NAME}/.bashrc
  eval "$(gh completion -s bash)"
  if [ "$SHLVL" = 2 ]; then
    script --flush ~/bashlog/script/`date "+%Y%m%d%H%M%S"`.log
  fi
  export PROMPT_COMMAND='history -a'
  export PS4='+\e[32m[\t]\e[34m[${BASH_SOURCE[0]##*/}:${LINENO}]\e[m> '
  export HISTFILE=~/bashlog/.bash_history
 EOT
 mkdir -p /home/${USER_NAME}/bashlog/script
 touch /home/${USER_NAME}/bashlog/.bash_history
EOF

RUN mkdir -p /home/${USER_NAME}/.vscode-server/extensions

EXPOSE 3000 9229

CMD ["yarn", "dev"]

```

コンテナ内で確認した結果。  

```bash
$ apt list --installed | grep git

WARNING: apt does not have a stable CLI interface. Use with caution in scripts.

findutils/now 4.6.0+git+20190209-2 arm64 [installed,local]
git-man/now 1:2.20.1-2+deb10u3 all [installed,local]
git/now 1:2.20.1-2+deb10u4 arm64 [installed,local]
librtmp1/now 2.4+20151223.gitfa8646d.1-2 arm64 [installed,local]
libtiff-dev/now 4.1.0+git191117-2~deb10u4 arm64 [installed,local]
libtiff5/now 4.1.0+git191117-2~deb10u4 arm64 [installed,local]
libtiffxx5/now 4.1.0+git191117-2~deb10u4 arm64 [installed,local]
```

上記のとおり`git`が再インストールされている。
