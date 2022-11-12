# Gitを最新版にする際のDockerfileの記述

## 記述

以下のように記述する。  

```Dockerfile
# syntax=docker/dockerfile:1


ARG GIT_VERSION=2.38.1

RUN <<-EOF
  set -e
  rm -f /etc/apt/apt.conf.d/docker-clean
  cat <<-'EOT' > /etc/apt/apt.conf.d/keep-cache
    Binary::apt::APT::Keep-Downloaded-Packages "true";
  EOT
EOF
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
--mount=type=cache,target=/var/lib/apt,sharing=locked <<-EOF
  set -eu
  apt-get update -qq
  apt-get install -qy gettext libcurl4-gnutls-dev
  cd /usr/local/src
  wget https://github.com/git/git/archive/refs/tags/v${GIT_VERSION}.tar.gz
  tar -xzf v${GIT_VERSION}.tar.gz
  cd git-${GIT_VERSION}
  make prefix=/usr/local all -j "$(nproc)"
  make prefix=/usr/local install
EOF
```

## 解説

```Dockerfile
RUN <<-EOF
  set -e
  rm -f /etc/apt/apt.conf.d/docker-clean
  cat <<-'EOT' > /etc/apt/apt.conf.d/keep-cache
    Binary::apt::APT::Keep-Downloaded-Packages "true";
  EOT
EOF
RUN --mount=type=cache,target=/var/cache/apt,sharing=locked \
--mount=type=cache,target=/var/lib/apt,sharing=locked <<-EOF
```

上記はDockerの`mount=type=cache`を使用するための記述。  
<https://github.com/moby/buildkit/blob/master/frontend/dockerfile/docs/reference.md#run---mounttypecache>

```Dockerfile
  apt-get update -qq
  apt-get install -qy gettext libcurl4-gnutls-dev
  cd /usr/local/src
  wget https://github.com/git/git/archive/refs/tags/v${GIT_VERSION}.tar.gz
  tar -xzf v${GIT_VERSION}.tar.gz
  cd git-${GIT_VERSION}
  make prefix=/usr/local all -j "$(nproc)"
  make prefix=/usr/local install
EOF
```

上記はGitをソースからインストールするための記述。  
<https://git-scm.com/book/ja/v2/%E4%BD%BF%E3%81%84%E5%A7%8B%E3%82%81%E3%82%8B-Git%E3%81%AE%E3%82%A4%E3%83%B3%E3%82%B9%E3%83%88%E3%83%BC%E3%83%AB>
