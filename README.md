[![Build Status](https://travis-ci.com/tfutils/kzenv.svg?branch=master)](https://travis-ci.com/tfutils/tfenv)

# kzenv

[Kustomize](https://www.kustomize.io/) version manager inspired by [kzenv](https://github.com/rbenv/rbenv)

## Support

Currently kzenv supports the following OSes

- Mac OS X (64bit)
- Linux
  - 64bit
  - Arm

## Installation

### Automatic

Install via Homebrew

```console
$ brew install kzenv
```

### Manual

1. Check out kzenv into any path (here is `${HOME}/.kzenv`)

```console
$ git clone https://github.com/tfutils/kzenv.git ~/.kzenv
```

2. Add `~/.kzenv/bin` to your `$PATH` any way you like

```console
$ echo 'export PATH="$HOME/.kzenv/bin:$PATH"' >> ~/.bash_profile
```

OR you can make symlinks for `kzenv/bin/*` scripts into a path that is already added to your `$PATH` (e.g. `/usr/local/bin`) `OSX/Linux Only!`

```console
$ ln -s ~/.kzenv/bin/* /usr/local/bin
```

On Ubuntu/Debian touching `/usr/local/bin` might require sudo access, but you can create `${HOME}/bin` or `${HOME}/.local/bin` and on next login it will get added to the session `$PATH`
or by running `. ${HOME}/.profile` it will get added to the current shell session's `$PATH`.

```console
$ mkdir -p ~/.local/bin/
$ . ~/.profile
$ ln -s ~/.kzenv/bin/* ~/.local/bin
$ which kzenv
```

## Usage

### kzenv install [version]

Install a specific version of kustomize. Available options for version:

- `i.j.k` exact version to install
- `latest` is a syntax to install latest version
- `latest:<regex>` is a syntax to install latest version matching regex (used by grep -e)
- `min-required` is a syntax to recursively scan your kustomize files to detect which version is minimally required. See [required_version](https://www.kustomize.io/docs/configuration/kustomize.html) docs. Also [see min-required](#min-required) section below.

```console
$ kzenv install 0.7.0
$ kzenv install latest
$ kzenv install latest:^0.8
$ kzenv install min-required
$ kzenv install
```

If `shasum` is present in the path, kzenv will verify the download against Hashicorp's published sha256 hash.
If [keybase](https://keybase.io/) is available in the path it will also verify the signature for those published hashes using Hashicorp's published public key.

You can opt-in to using GnuPG tools for PGP signature verification if keybase is not available:

```console
$ echo 'trust-kzenv: yes' > ~/.tfenv/use-gpgv
$ kzenv install
```

The `trust-kzenv` directive means that verification uses a copy of the
Hashicorp OpenPGP key found in the kzenv repository. Skipping that directive
means that the Hashicorp key must be in the existing default trusted keys.
Use the file `~/.kzenv/use-gnupg` to instead invoke the full `gpg` tool and
see web-of-trust status; beware that a lack of trust path will not cause a
validation failure.

#### .kustomize-version

If you use a [.kustomize-version file](#kustomize-version-file), `kzenv install` (no argument) will install the version written in it.

#### min-required

Please note that we don't do semantic version range parsing but use first ever found version as the candidate for minimally required one. It is up to the user to keep the definition reasonable. I.e.

```kustomize
// this will detect 0.12.3
kustomize {
  required_version  = "<0.12.3, >= 0.10.0"
}
```

```kustomize
// this will detect 0.10.0
kustomize {
  required_version  = ">= 0.10.0, <0.12.3"
}
```

### Environment Variables

#### kzenv

##### `kzenv_ARCH`

String (Default: amd64)

Specify architecture. Architecture other than the default amd64 can be specified with the `kzenv_ARCH` environment variable

```console
kzenv_ARCH=arm tfenv install 0.7.9
```

##### `kzenv_CURL_OUTPUT`

Integer (Default: 2)

Set the mechanism used for displaying download progress when downloading kustomize versions from the remote server.

- 2: v1 Behaviour: Pass `-#` to curl
- 1: Use curl default
- 0: Pass `-s` to curl

##### `kzenv_DEBUG`

##### `kzenv_REMOTE`

String (Default: https://releases.hashicorp.com)

To install from a remote other than the default

```console
kzenv_REMOTE=https://example.jfrog.io/artifactory/hashicorp
```

#### Bashlog Logging Library

##### `BASHLOG_COLOURS`

Integer (Default: 1)

To disable colouring of console output, set to 0.

##### `BASHLOG_DATE_FORMAT`

String (Default: +%F %T)

The display format for the date as passed to the `date` binary to generate a datestamp used as a prefix to:

- `FILE` type log file lines.
- Each console output line when `BASHLOG_EXTRA=1`

##### `BASHLOG_EXTRA`

Integer (Default: 0)

By default, console output from kzenv does not print a date stamp or log severity.

To enable this functionality, making normal output equivalent to FILE log output, set to 1.

##### `BASHLOG_FILE`

Integer (Default: 0)

Set to 1 to enable plain text logging to file (FILE type logging).

The default path for log files is defined by /tmp/$(basename $0).log
Each executable logs to its own file.

e.g.

```console
BASHLOG_FILE=1 kzenv use latest
```

will log to `/tmp/kzenv-use.log`

##### `BASHLOG_FILE_PATH`

String (Default: /tmp/$(basename ${0}).log)

To specify a single file as the target for all FILE type logging regardless of the executing script.

##### `BASHLOG_I_PROMISE_TO_BE_CAREFUL_CUSTOM_EVAL_PREFIX`

String (Default: "")

_BE CAREFUL - MISUSE WILL DESTROY EVERYTHING YOU EVER LOVED_

This variable allows you to pass a string containing a command that will be executed using `eval` in order to produce a prefix to each console output line, and each FILE type log entry.

e.g.

```console
BASHLOG_I_PROMISE_TO_BE_CAREFUL_CUSTOM_EVAL_PREFIX='echo "${$$} "'
```

will prefix every log line with the calling process' PID.

##### `BASHLOG_JSON`

Integer (Default: 0)

Set to 1 to enable JSON logging to file (JSON type logging).

The default path for log files is defined by /tmp/$(basename $0).log.json
Each executable logs to its own file.

e.g.

```console
BASHLOG_JSON=1 kzenv use latest
```

will log in JSON format to `/tmp/kzenv-use.log.json`

JSON log content:

`{"timestamp":"<date +%s>","level":"<log-level>","message":"<log-content>"}`

##### `BASHLOG_JSON_PATH`

String (Default: /tmp/$(basename ${0}).log.json)

To specify a single file as the target for all JSON type logging regardless of the executing script.

##### `BASHLOG_SYSLOG`

Integer (Default: 0)

To log to syslog using the `logger` binary, set this to 1.

The basic functionality is thus:

```console
local tag="${BASHLOG_SYSLOG_TAG:-$(basename "${0}")}";
local facility="${BASHLOG_SYSLOG_FACILITY:-local0}";
local pid="${$}";

logger --id="${pid}" -t "${tag}" -p "${facility}.${severity}" "${syslog_line}"
```

##### `BASHLOG_SYSLOG_FACILITY`

String (Default: local0)

The syslog facility to specify when using SYSLOG type logging.

##### `BASHLOG_SYSLOG_TAG`

String (Default: $(basename $0))

The syslog tag to specify when using SYSLOG type logging.

Defaults to the PID of the calling process.

### kzenv use &lt;version>

Switch a version to use

`latest` is a syntax to use the latest installed version

`latest:<regex>` is a syntax to use latest installed version matching regex (used by grep -e)

`min-required` will switch to the version minimally required by your kustomize sources (see above `kzenv install`)

```console
$ kzenv use min-required
$ kzenv use 0.7.0
$ kzenv use latest
$ kzenv use latest:^0.8
```

### kzenv uninstall &lt;version>

Uninstall a specific version of kustomize
`latest` is a syntax to uninstall latest version
`latest:<regex>` is a syntax to uninstall latest version matching regex (used by grep -e)

```console
$ kzenv uninstall 0.7.0
$ kzenv uninstall latest
$ kzenv uninstall latest:^0.8
```

### kzenv list

List installed versions

```console
% kzenv list
* 0.10.7 (set by /opt/kzenv/version)
  0.9.0-beta2
  0.8.8
  0.8.4
  0.7.0
  0.7.0-rc4
  0.6.16
  0.6.2
  0.6.1
```

### kzenv list-remote

List installable versions

```console
% kzenv list-remote
0.9.0-beta2
0.9.0-beta1
0.8.8
0.8.7
0.8.6
0.8.5
0.8.4
0.8.3
0.8.2
0.8.1
0.8.0
0.8.0-rc3
0.8.0-rc2
0.8.0-rc1
0.8.0-beta2
0.8.0-beta1
0.7.13
0.7.12
...
```

## .kustomize-version file

If you put a `.kustomize-version` file on your project root, or in your home directory, kzenv detects it and uses the version written in it. If the version is `latest` or `latest:<regex>`, the latest matching version currently installed will be selected.

```console
$ cat .kustomize-version
0.6.16

$ kustomize --version
kustomize v0.6.16


$ echo 0.7.3 > .kustomize-version

$ kustomize --version
kustomize v0.7.3

$ echo latest:^0.8 > .kustomize-version

$ kustomize --version
kustomize v0.8.8
```

## Upgrading

```console
$ git --git-dir=~/.kzenv/.git pull
```

## Uninstalling

```console
$ rm -rf /some/path/to/kzenv
```

## LICENSE

- [kzenv itself](https://github.com/tfutils/tfenv/blob/master/LICENSE)
- [rbenv](https://github.com/rbenv/rbenv/blob/master/LICENSE)
  - kzenv partially uses rbenv's source code
