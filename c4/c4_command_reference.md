---
tool_name: c4
doc_type: reference
category: c4 Reference
title: "c4 Command and Flag Reference"
summary: "Complete top-level c4 command list with flags and key subcommands"
---
# C4 Command and Flag Reference

Last verified: 2026-03-04
Verified host: `ddaka@10.70.0.171`
CLI version: `4.24.3`

## Global flags

All top-level commands support:

```text
-h, --help
    --loglevel string
-v, --version   (top-level c4 only)
```

## Top-level commands

- `arch`
- `aws`
- `awscreds`
- `azure`
- `completion`
- `config`
- `connect`
- `diag`
- `down`
- `fetch`
- `gcp`
- `help`
- `host`
- `info`
- `initialize`
- `jq`
- `local`
- `mount`
- `pigz`
- `play`
- `ps`
- `pwdhash`
- `rm`
- `semver`
- `server`
- `sqlclient`
- `tag`
- `template`
- `up`
- `update`
- `upload`
- `version`
- `wait`
- `yq`

## Commands and flags

### `c4 arch`

- Flags: `-h, --help`

### `c4 aws`

- Subcommands: `add`, `diag`, `down`, `ebsnvme-id`, `info`, `play`, `ps`, `reboot`, `reconf`, `rm`, `run`, `ssh`, `start`, `start-session`, `tags`, `token`, `up`, `vscode`, `wait`
- Flags:
  - `-C, --connect`
  - `-h, --help`
  - `-n, --nodes strings`
  - `-o, --output string`
  - `-q, --quiet`
  - `-T, --run-template`
  - `-U, --show-play-id` (deprecated)
  - `--show-template`
  - `--wait-stage string`

### `c4 awscreds`

- Flags:
  - `-h, --help`
  - `-j, --json`

### `c4 azure`

- Subcommands: `add`, `diag`, `down`, `info`, `play`, `ps`, `reboot`, `reconf`, `rm`, `run`, `ssh`, `start`, `up`, `vscode`, `wait`
- Flags: same as `c4 aws`

### `c4 completion`

- Subcommands: `bash`, `fish`, `powershell`, `zsh`
- Flags: `-h, --help`

### `c4 config`

- Flags:
  - `-K, --apropos string`
  - `-e, --eval string`
  - `--explain`
  - `-E, --exported`
  - `-f, --from string`
  - `-F, --fullconfig`
  - `-g, --grep string`
  - `-G, --ground`
  - `-h, --help`
  - `-H, --help-on module[:usage]`
  - `--highlight string`
  - `-i, --input strings`
  - `-I, --input-raw stringArray`
  - `-k, --key string`
  - `--no-ground`
  - `-n, --node string`
  - `-R, --raw`
  - `-S, --subtract strings`
  - `-t, --to string`
  - `-V, --validate`
  - `--validators`

### `c4 connect`

- Flags:
  - `-C, --enable-completion`
  - `-F, --force-pty`
  - `-h, --help`
  - `-H, --host string`
  - `--internal`
  - `-N, --no-command`
  - `-T, --no-pty`
  - `--no-tunnel`
  - `-n, --node uint`
  - `-o, --output string`
  - `-i, --play-id string`
  - `-p, --port int`
  - `-q, --quiet`
  - `-e, --send-env strings`
  - `-S, --simple`
  - `-s, --subsystem string`
  - `-t, --target string`
  - `-w, --wait int`

### `c4 diag`

- Flags:
  - `-h, --help`
  - `-i, --input string`
  - `-l, --list`

### `c4 down`

- Flags:
  - `-h, --help`
  - `-n, --nodes strings`
  - `-r, --remote string`

### `c4 fetch`

- Flags:
  - `-a, --all`
  - `-h, --help`
  - `-L, --list-fetched`
  - `--local`
  - `-p, --public`
  - `-R, --recursive`
  - `-r, --refetch`
  - `-T, --test`
  - `--to string`
  - `-u, --upload`
  - `--when string`

### `c4 gcp`

- Subcommands: `add`, `diag`, `down`, `info`, `play`, `ps`, `reboot`, `reconf`, `rm`, `run`, `ssh`, `start`, `up`, `vscode`, `wait`
- Flags: same as `c4 aws`

### `c4 help`

- Flags:
  - `-h, --help`
  - `--style string` (`man`, `cobra`)

### `c4 host`

- Subcommands: `add`, `diag`, `down`, `info`, `play`, `reboot`, `reconf`, `reserve`, `rm`, `run`, `ssh`, `start`, `up`, `vscode`, `wait`
- Flags: same as `c4 aws`

### `c4 info`

- Flags:
  - `-h, --help`
  - `-i, --input`
  - `-j, --jq`
  - `-L, --list`
  - `--local`
  - `-p, --public`
  - `-R, --recursive`
  - `-r, --remote`

### `c4 initialize`

- Subcommands: `done`, `wait`
- Direct flags: none listed in help output

### `c4 jq`

- Flags:
  - `-c, --compact-output`
  - `-e, --exit-status`
  - `-h, --help`
  - `-r, --raw-output`
  - `-s, --slurp`
  - `--var stringArray`

### `c4 local`

- Subcommands: `add`, `down`, `info`, `play`, `reboot`, `reconf`, `rm`, `run`, `ssh`, `start`, `up`, `vscode`, `wait`
- Flags: same as `c4 aws`

### `c4 mount`

- Flags:
  - `--fakeroot string`
  - `-k, --fakeroottrue`
  - `-f, --fetch`
  - `-h, --help`
  - `--local`
  - `-x, --nsexec`
  - `-T, --test`
  - `-e, --with-env`
  - `-c, --working-copy string`

### `c4 pigz`

- Flags:
  - `-d, --decompress`
  - `-f, --force`
  - `-h, --help`
  - `-c, --stdout`
  - `-S, --suffix string`

### `c4 play`

This command uses legacy `AVAILABLE OPTIONS` in help output:

- `-D`
- `-F`
- `-I INSTANCE-ID`
- `-N NUMBER`
- `-X`
- `-d DISKS`
- `-i NODE-ID`
- `-n NUMBER`
- `-v FROM:TO`

### `c4 ps`

- Flags:
  - `-a, --all`
  - `-A, --all-owners`
  - `-C, --columns strings`
  - `-e, --eval string`
  - `--expired`
  - `-h, --help`
  - `--internal`
  - `--latest`
  - `--latest-full`
  - `--list-known-columns`
  - `--local`
  - `-m, --metadata`
  - `-o, --output string`
  - `-q, --quiet`
  - `--remote-timeout int`
  - `-s, --short`
  - `-S, --style string`
  - `-t, --tele`
  - `--user`
  - `-V, --views strings`

### `c4 pwdhash`

- Flags:
  - `-c, --crypt`
  - `-h, --help`
  - `-i, --stdin`
  - `-V, --verify`

### `c4 rm`

This command uses legacy `AVAILABLE OPTIONS` in help output:

- `-d`
- `-f`
- `-g`
- `-n`
- `-p`
- `-w`

### `c4 semver`

- Flags:
  - `-c, --constraint string`
  - `-e, --expr string`
  - `-h, --help`
  - `-s, --sort`

### `c4 server`

- Flags:
  - `--certFile string`
  - `-d, --disable-auth`
  - `-t, --disable-tls`
  - `-h, --help`
  - `--keyFile string`
  - `-m, --mdns`
  - `--passwdFile string`
  - `--port string`
  - `-s, --socket string`
  - `-e, --sse`

### `c4 sqlclient`

- Flags:
  - `-c, --connection string`
  - `-h, --help`
  - `-T, --notls`
  - `-p, --password string`
  - `-q, --query string`
  - `-s, --skiptlsverify`
  - `-S, --tlsverify`
  - `-u, --user string`
  - `-t, --usetls`

### `c4 tag`

This command uses legacy `AVAILABLE OPTIONS` in help output:

- `-P`
- `-V TEMPLATE`
- `-f`
- `-l [FILTER]`
- `-p`
- `-r ALIAS`

### `c4 template`

- Subcommands: `render`
- Flags:
  - `-c, --config stringArray`
  - `--embedded`
  - `-e, --entry string`
  - `-h, --help`
  - `-o, --out string`
  - `-T, --template string`
  - `-t, --type string`

### `c4 up`

- Flags:
  - `-h, --help`
  - `-n, --nodes strings`
  - `-r, --remote string`

### `c4 update`

This command uses legacy `AVAILABLE OPTIONS` in help output:

- `-A`
- `-P`
- `-a`
- `-f`
- `-p ID[@IP]`
- `-t TARGET`
- `-v`

### `c4 upload`

- Flags:
  - `--fallback-to-private`
  - `-h, --help`
  - `--local`
  - `-p, --public`
  - `-P, --public-only`

### `c4 version`

- Flags:
  - `-h, --help`
  - `--info`
  - `--licenses`

### `c4 wait`

- Flags:
  - `-c, --conditions strings`
  - `-o, --conditions-logic-or`
  - `-h, --help`
  - `-n, --nodes strings`
  - `-r, --remote string`
  - `-s, --stage string`

### `c4 yq`

- Flags:
  - `-c, --compact`
  - `-h, --help`
  - `-r, --raw-output`
  - `-Y, --yaml`

## Notes

- For deep subcommand details (for example `c4 aws play`, `c4 host reconf`, `c4 template render`), use `c4 <command> <subcommand> --help`.
- Cloud wrappers (`aws`, `azure`, `gcp`) and runtime wrappers (`host`, `local`) share a common top-level flag pattern.
```

## c4 /usr/sbin/universalaccessd
```text
```

## c4 /sbin/dmesg
```text
```

## c4 /usr/bin/xsubpp5.34
```text
```

## c4 /usr/bin/python3
```text
```

## c4 /usr/bin/snmpget
```text
```

## c4 /usr/bin/dispqlen.d
```text
```

## c4 /usr/bin/pod2text
```text
```

## c4 /usr/sbin/kdcsetup
```text
```

## c4 /sbin/mount_virtiofs
```text
```

## c4 /usr/bin/mailq
```text
```

## c4 /usr/bin/llvm-g++
```text
```

## c4 /usr/bin/sc_usage
```text
```

## c4 /usr/sbin/localemanager
```text
```

## c4 /usr/sbin/purge
```text
```

## c4 /usr/bin/jhsdb
```text
```

## c4 /usr/sbin/smbdiagnose
```text
```

## c4 /usr/bin/genstrings
```text
```

## c4 /usr/bin/g++
```text
```

## c4 /usr/bin/base64
```text
```

## c4 /usr/sbin/cvadmin
```text
```

## c4 /usr/bin/mailx
```text
```

## c4 /usr/bin/corelist5.34
```text
```

## c4 /usr/sbin/dbmmanage
```text
```
