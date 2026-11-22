# Semaphore UI Homebrew Tap

Homebrew formula for [Semaphore UI](https://semaphoreui.com): run Ansible, Terraform,
OpenTofu, PowerShell and Bash tasks from a web UI, as a background service on macOS.

## Install

```sh
brew install semaphoreui/tap/semaphore
```

Or `brew tap semaphoreui/tap` and then `brew install semaphore`.

The formula builds Semaphore from source (Go backend and Vue frontend), so the first
install takes a few minutes unless a bottle is available.

## First run

Installation creates a config with freshly generated encryption keys and an SQLite
database location:

| Path                                          | Contents                       |
|-----------------------------------------------|--------------------------------|
| `$(brew --prefix)/etc/semaphore/config.json`  | Config (mode `0600`)           |
| `$(brew --prefix)/var/semaphore/`             | SQLite database, cloned repos  |
| `$(brew --prefix)/var/log/semaphore.log`      | Service log                    |

An existing config is never overwritten, so upgrades keep your settings and keys.
Back up `config.json`: without `access_key_encryption` the keys and secrets stored in
the database can't be decrypted.

Create the first admin user:

```sh
semaphore user add --admin \
  --login admin --name Admin --email admin@localhost --password '<password>' \
  --config "$(brew --prefix)/etc/semaphore/config.json"
```

Start the service and open <http://127.0.0.1:3000>:

```sh
brew services start semaphore
```

Install the tools your templates run, for example:

```sh
brew install ansible opentofu
```

The service runs with Homebrew's `bin` directories on `PATH`, so tools installed with
Homebrew are found automatically.

## Managing the service

```sh
brew services info semaphore      # status
brew services restart semaphore   # after editing config.json
brew services stop semaphore
tail -f "$(brew --prefix)/var/log/semaphore.log"
```

To run Semaphore in the foreground instead:

```sh
semaphore server --config "$(brew --prefix)/etc/semaphore/config.json"
```

## Configuration

The generated config binds to `127.0.0.1:3000`. To make Semaphore reachable from
other machines, change `"interface"` (for example to `"0.0.0.0"`) and restart the
service. Set `"web_host"` to the public URL when running behind a reverse proxy.

To use MySQL or PostgreSQL instead of SQLite, replace the `"dialect"` and `"sqlite"`
keys, for example:

```json
"dialect": "postgres",
"postgres": {
  "host": "127.0.0.1:5432",
  "user": "semaphore",
  "pass": "<password>",
  "name": "semaphore",
  "options": { "sslmode": "disable" }
}
```

All options are described in the
[configuration docs](https://docs.semaphoreui.com/administration-guide/configuration/).
Most of them can also be set with `SEMAPHORE_*` environment variables.

To regenerate the default config, remove it and rerun the post-install step:

```sh
rm "$(brew --prefix)/etc/semaphore/config.json"
brew postinstall semaphore
```

## Uninstall

```sh
brew services stop semaphore
brew uninstall semaphore
rm -rf "$(brew --prefix)/etc/semaphore" "$(brew --prefix)/var/semaphore"
```

## Development

```sh
brew style semaphoreui/tap/semaphore
brew audit --strict semaphoreui/tap/semaphore
brew install --build-from-source semaphoreui/tap/semaphore
brew test semaphoreui/tap/semaphore
```

Pull requests are built and tested on macOS by `brew test-bot`. To publish the bottles
of a green pull request, run the `brew pr-pull` workflow with its number. The
`brew bump` workflow opens a pull request when a new Semaphore release is out.
