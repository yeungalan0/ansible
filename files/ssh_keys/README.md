# SSH Public Keys

Place your SSH public key files (`.pub`) in this directory.
All `*.pub` files found here will be deployed to the Pi user's
`~/.ssh/authorized_keys` when the Raspberry Pi playbook runs.

## Example

```bash
cp ~/.ssh/id_ed25519.pub files/ssh_keys/my_key.pub
```

**Do NOT place private keys in this directory or anywhere in this repo.**
