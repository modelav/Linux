# SSH keys

SSH public-key authentication lets a client authenticate to a server using a key pair. The private key stays on the client; the matching public key is installed in the account's `authorized_keys` file on the server.

## Generate a key pair

Run this on the client machine, not on the server you want to access:

```bash
ssh-keygen -t ed25519 -C "admin-laptop"
```

`ssh-keygen` creates a private key and a `.pub` public-key file. By default, it proposes a path under `~/.ssh/` and prompts for a passphrase. Use a passphrase to protect the private key if it is copied or stolen.

| Option | Meaning |
|---|---|
| `-t <type>` | Choose the key type, for example `ed25519`. |
| `-C <comment>` | Add a comment to identify the key. |
| `-f <path>` | Choose the output key-file path. |
| `-a <rounds>` | Set the number of KDF rounds used to protect a passphrase-protected private key. |
| `-l` | Display the fingerprint of the key file specified with `-f`. |
Example with a custom filename:

```bash
ssh-keygen -t ed25519 -a 64 -C "admin-laptop" -f ~/.ssh/debian_admin
```

Never copy or share the private key file. The public key is the file ending in `.pub`.

## Install the public key on a server

The simplest method is `ssh-copy-id`, run from the client:

```bash
ssh-copy-id -i ~/.ssh/id_ed25519.pub alice@server.example.com
```

It connects to the server and appends the public key to the remote user's `~/.ssh/authorized_keys` file. It usually requires an existing way to log in, such as a password or another key.

You can also install the public key manually by appending its one-line contents to:

```text
/home/alice/.ssh/authorized_keys
```


## Check SSH key permissions

On the server, the SSH directory and authorized keys file should be protected and owned by the account:

```bash
sudo chown -R alice:alice /home/alice/.ssh
sudo chmod 700 /home/alice/.ssh
sudo chmod 600 /home/alice/.ssh/authorized_keys
```

Adjust the home path and account name for the server user. The private key on the client should also be readable only by its owner:

```bash
chmod 600 ~/.ssh/id_ed25519
```


## Connect using a key

```bash
ssh alice@server.example.com
ssh -i ~/.ssh/debian_admin alice@server.example.com
```

Use `-i` to specify a particular private key.

## Manage keys with ssh-agent

`ssh-agent` can hold unlocked private keys in memory so you do not need to enter the key passphrase repeatedly during the agent's lifetime.

```bash
ssh-add ~/.ssh/id_ed25519
ssh-add -l
ssh-add -t 1h ~/.ssh/id_ed25519
ssh-add -d ~/.ssh/id_ed25519
ssh-add -D
```

| Option | Meaning |
|---|---|
| `-l` | List fingerprints of identities held by the agent. |
| `-t <time>` | Set an identity lifetime. |
| `-d <file>` | Remove the specified identity. |
| `-D` | Remove all identities from the agent. |

An agent must be running and its environment variables must be available to the shell.

## Check key fingerprints

```bash
ssh-keygen -lf ~/.ssh/id_ed25519.pub
```


## Troubleshooting

Use verbose client output to see which keys and authentication methods are being tried:

```bash
ssh -v alice@server.example.com
ssh -vvv alice@server.example.com
```

On the server, check:

- The public key is in the correct user's `~/.ssh/authorized_keys` file.
- The home directory, `.ssh`, and `authorized_keys` ownership and permissions are acceptable.
- The account is allowed to log in and has a usable shell, if an interactive shell is expected.
- The SSH server configuration allows public-key authentication.
- Server logs contain no related authentication or permission errors; check them with `sudo journalctl -u ssh`.

Do not disable password authentication until key-based login has been tested in a separate session and a recovery path is available.
