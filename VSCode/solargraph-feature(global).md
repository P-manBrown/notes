# Solargraph(devcontainer-feature)の環境変数対応版

## `devcontainer-feature.json`

`./src/solargraph/devcontainer-feature.json`

```json
{
  "id": "solargraph",
  "version": "1.0.0",
  "name": "Solargraph",
  "documentationURL": "https://github.com/P-manBrown/devcontainer-features/tree/main/src/solargraph",
  "licenseURL": "https://github.com/P-manBrown/devcontainer-features/blob/main/LICENSE",
  "description": "Install Solargraph.",
  "options": {
    "installSolargraphRails": {
      "type": "boolean",
      "default": "true",
      "description": "Install solargraph-rails?"
    },
    "skipYardGems": {
      "type": "boolean",
      "default": "false",
      "description": "Skip running `yard gems`?"
    },
    "solargraphVersion": {
      "type": "string",
      "proposals": ["latest"],
      "default": "latest",
      "description": "Enter a Solargraph version."
    },
    "solargraphRailsVersion": {
      "type": "string",
      "proposals": ["latest"],
      "default": "latest",
      "description": "Enter a solargraph-rails version."
    },
    "solargraphCache": {
      "type": "string",
      "default": "${_REMOTE_USER_HOME}/.solargraph/cache",
      "description": "Enter the Solargraph cache directory path."
    },
    "solargraphGlobalConfig": {
      "type": "string",
      "default": "${_REMOTE_USER_HOME}/.config/solargraph/config.yml",
      "description": "Enter the Solargraph global config file path."
    }
  },
  "entrypoint": "/usr/local/share/solargraph-env.sh",
  "installsAfter": [
    "ghcr.io/devcontainers/features/common-utils",
    "ghcr.io/devcontainers/features/ruby"
  ]
}

```

## `install.sh`

`./src/solargraph/install.sh`

```sh
#!/usr/bin/env bash
set -eu

INSTALL_SOLARGRAPH_RAILS="${INSTALLSOLARGRAPHRAILS}"
SKIP_YARD_GEMS="${SKIPYARDGEMS}"
SOLARGRAPH_VERSION="${SOLARGRAPHVERSION}"
SOLARGRAPH_RAILS_VERSION="${SOLARGRAPHRAILSVERSION}"
SOLARGRAPH_GLOBAL_CONFIG="${SOLARGRAPHGLOBALCONFIG}"
export SOLARGRAPH_CACHE="${SOLARGRAPHCACHE}"

USER_NAME="${_REMOTE_USER}"
USER_HOME="${_REMOTE_USER_HOME}"

err() {
  printf '\e[31m%s\e[m\n' "$*" >&2
}

retry() {
  local cmd_status=1
  local retry_count=0
  local max_retry=3
  set +e
    until [[ ${cmd_status} -eq 0 ]] || [[ ${retry_count} -eq ${max_retry} ]]; do
      echo "Retry count: ${retry_count}"
      "$@"
      cmd_status=$?
      (( retry_count++ ))
      sleep 1
    done
  set -e
}

# Check
## User
if [[ "$(id -u)" -ne 0 ]]; then
  message="$(
    cat <<-EOF
    --------------------------------------------------------
      Script must be run as root.
      Use sudo, su, or add "USER root" to your Dockerfile.
    --------------------------------------------------------
    EOF
  )"
  err "${message}"
  exit 1
fi
## RubyGems
if ! ruby --version > /dev/null 2>&1; then
  err "ERROR: Install Ruby before running this script."
  exit 1
fi

# Install Gems
## solargraph
echo 'Installing solargraph...'
if [[ "${SOLARGRAPH_VERSION}" == 'latest' ]]; then
  gem install solargraph
else
  gem install solargraph --version "${SOLARGRAPH_VERSION}"
fi
## solargraph-rails
if [[ "${INSTALL_SOLARGRAPH_RAILS}" == 'true' ]]; then
  echo 'Installing solargraph-rails...'
  if [[ "${SOLARGRAPH_RAILS_VERSION}" == 'latest' ]]; then
    gem install solargraph-rails
  else
    gem install solargraph-rails --version "${SOLARGRAPH_RAILS_VERSION}"
  fi
fi

# Set up solargraph
echo 'Setting up Solargraph...'
(
  HOME="${USER_HOME}"
  solargraph download-core
  touch "${HOME}/.gemrc"
  yard config --gem-install-yri
)
if [[ "${SKIP_YARD_GEMS}" == 'false' ]]; then
  retry yard gems --quiet
fi
if [[ ! -e "${SOLARGRAPH_GLOBAL_CONFIG}" ]]; then
  mkdir -pv "${SOLARGRAPH_GLOBAL_CONFIG%/*}" > ./made_dir_list.txt
  solargraph config "${SOLARGRAPH_GLOBAL_CONFIG%/*}"
  mv \
    "${SOLARGRAPH_GLOBAL_CONFIG%/*}/.solargraph.yml" \
    "${SOLARGRAPH_GLOBAL_CONFIG}"
fi
if [[ "${INSTALL_SOLARGRAPH_RAILS}" == 'true' ]] \
  && ! grep -q 'solargraph-rails' "${SOLARGRAPH_GLOBAL_CONFIG}"; then
    sed -i \
    '/plugins:/s/\[]//; /plugins:/a - solargraph-rails' \
    "${SOLARGRAPH_GLOBAL_CONFIG}"
fi

# Change owner
if [[ "${USER_NAME}" != "root" ]]; then
  gem_dir="$(gem environment gemdir)"
  chown -R "${USER_NAME}" "${gem_dir}"
  solargraph_cache="${SOLARGRAPH_CACHE:-${USER_HOME}/.solargraph}"
  chown -R "${USER_NAME}" "${solargraph_cache}"
  chown "${USER_NAME}" "${USER_HOME}/.gemrc"
  solargraph_global_config_dir="$(
    head -1 ./made_dir_list.txt \
    | grep -Eo "'.+'" \
    | tr -d "'"
  )"
  if [[ -n "${solargraph_global_config_dir}" ]]; then
    chown -R "${USER_NAME}" "${solargraph_global_config_dir}"
  else
    chown "${USER_NAME}" "${SOLARGRAPH_GLOBAL_CONFIG}"
  fi
fi

echo "Done!!"

```

## `scenarios.json`

`./test/solargraph/scenarios.json`

```json
{
  "solargraph-test": {
    "image": "mcr.microsoft.com/devcontainers/ruby",
    "remoteUser": "vscode",
    "remoteEnv": {
      "SOLARGRAPH_GLOBAL_CONFIG": "${localEnv:SOLARGRAPH_GLOBAL_CONFIG}",
      "SOLARGRAPH_CACHE": "${localEnv:SOLARGRAPH_CACHE}"
    },
    "features": {
      "solargraph": {
        "skipYardGems": true,
        "solargraphGlobalConfig": "${localEnv:SOLARGRAPH_GLOBAL_CONFIG}",
        "solargraphCache": "${localEnv:SOLARGRAPH_CACHE}"
      }
    }
  }
}

```

## `solargraph-test.sh`

`./test/solargraph/solargraph-test.sh`

```sh
#!/bin/bash
set -e

# Optional: Import test library
source dev-container-features-test-lib

# Definition specific tests
check "solargraph_version" solargraph --version
check "solargraph-rails_version" gem list solargraph-rails
check "global_config_file" test -e "${SOLARGRAPH_GLOBAL_CONFIG}"
check "config_file_rails" grep 'solargraph-rails' "${SOLARGRAPH_GLOBAL_CONFIG}"
check "solargraph_cache" test -e "${SOLARGRAPH_CACHE}"

# Report result
reportResults

```

## 環境変数について

`SOLARGRAPH_GLOBAL_CONFIG`と`SOLARGRAPH_CACHE`は別途イメージ内に環境変数を設定しなければならない。  

`{remoteEnv: VARIABLE}`や`{containerEnv: VARIABLE}`はイメージ内に設定されないため使用不可。  
