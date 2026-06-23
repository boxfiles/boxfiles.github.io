# copy (/plugins/builtin/copy)



# `copy` [#copy]

> \[!WARNING]
> `copy` planning exists. Apply behavior is currently stubbed and does not mutate workstation state yet.

## Purpose [#purpose]

Copy a file or directory from the current manifest's sibling `files/` directory to a workstation target path.

## Manifest syntax [#manifest-syntax]

```yaml
steps:
  - id: copy-gitconfig
    uses: copy
    with:
      from: gitconfig
      to: ~/.gitconfig
      overwrite: false
```

For this manifest layout:

```text
modules/git.yaml
modules/files/gitconfig
```

`from: gitconfig` resolves to:

```text
<rootDir>/modules/files/gitconfig
```

## Config [#config]

| Field       | Type    | Required | Meaning                                          |
| ----------- | ------- | -------: | ------------------------------------------------ |
| `from`      | string  |      yes | Source path relative to `ctx.manifest.filesDir`. |
| `to`        | string  |      yes | Destination path on the workstation.             |
| `overwrite` | boolean |       no | Allows replacing existing destination content.   |

## Source path rules [#source-path-rules]

`from` is already relative to the manifest-local `files/` directory.

Valid:

```yaml
from: gitconfig
from: shell/zshrc
```

Invalid:

```yaml
from: files/gitconfig
from: ./files/gitconfig
from: ../secret
from: /etc/passwd
```

Boxfiles rejects invalid source prefixes during provider config validation.

## Planning behavior [#planning-behavior]

`copy` emits one planned change:

* `operation: create` when `overwrite` is false or omitted
* `operation: update` when `overwrite` is true
* `after.source` contains the resolved absolute source path

Safety:

* `idempotent: true`
* `unsafe: true` only when `overwrite: true`
* `reason: copy may overwrite existing target` when `overwrite: true`
