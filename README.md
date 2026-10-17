# pmctl — Passmower CLI

`pmctl` is a kubectl-compatible shell script for managing [Passmower](https://github.com/passmower/passmower) users, passwords, and group membership.

## Installation

```bash
curl -Lo ~/.local/bin/pmctl https://raw.githubusercontent.com/passmower/pmcli/main/pmctl
chmod +x ~/.local/bin/pmctl

# Optional: install as kubectl plugin
ln -s ~/.local/bin/pmctl ~/.local/bin/kubectl-passmower
# Then: kubectl passmower <command>
```

## Requirements

- `kubectl` configured with access to the cluster
- `jq` (for group management commands)
- For `set-password`: `apache2-utils` (provides `htpasswd`) or `python3-bcrypt`

## Usage

```
pmctl users                          # list all OIDCUsers
pmctl get <user>                     # show OIDCUser YAML
pmctl approve <user>                 # approve a user
pmctl unapprove <user>               # revoke approval

pmctl groups <user>                  # list groups
pmctl add-group <user> <prefix:name> # add a group
pmctl remove-group <user> <prefix:name>

pmctl set-password <user>            # set bcrypt password (interactive prompt)
pmctl remove-password <user>         # remove password Secret
```

## Environment variables

| Variable | Default | Purpose |
|---|---|---|
| `PASSMOWER_NAMESPACE` | current context namespace | namespace where Passmower is deployed |
| `GROUP_PREFIX` | `local` | default prefix for bare group names |
| `KUBECTL` | `kubectl` | kubectl binary path |

## Password login

`set-password` stores a bcrypt hash in a Kubernetes Secret named `passmower-password-<username>` in the Passmower namespace.  The server-side `PASSWORD_LOGIN_ENABLED` feature flag must be enabled in Passmower for the hash to be verified at login time.

Password hashing uses `htpasswd -nbBi` (from `apache2-utils`/`httpd-tools`) if available, falling back to `python3-bcrypt`. The password is never passed as a command-line argument.

## RBAC

The script needs:

```yaml
rules:
  - apiGroups: [codemowers.cloud]
    resources: [oidcusers]
    verbs: [get, list, patch]
  - apiGroups: [""]
    resources: [secrets]
    verbs: [get, create, apply, delete]
    # Ideally scoped with resourceNames or label selectors
```

## Related

- [Passmower](https://github.com/passmower/passmower) — the OIDC identity provider this tool manages
