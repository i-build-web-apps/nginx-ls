# nginx-ls

One command. Zero dependencies. Instant overview of your nginx routing.

`nginx-ls` reads your nginx config and shows you a clean table of every domain, where it listens, and where traffic goes. Works with local nginx, Docker containers, or piped from a remote host.

```
$ nginx-ls

DOMAIN                                LISTEN      TYPE    TARGET
------------------------------------  ----------  ------  -------------------------
example.com www.example.com           443 ssl     proxy   http://webapp:3000
api.example.com                       443 ssl     proxy   http://api-server:8080
docs.example.com                      443 ssl     static  /var/www/docs/public
staging.example.com                   443,80 ssl  proxy   http://staging-app:3000
```

## Install

```bash
# Download and run (quickest)
curl -fsSL https://raw.githubusercontent.com/i-build-web-apps/nginx-ls/main/nginx-ls | bash

# Install to PATH (permanent)
sudo curl -fsSL https://raw.githubusercontent.com/i-build-web-apps/nginx-ls/main/nginx-ls -o /usr/local/bin/nginx-ls
sudo chmod +x /usr/local/bin/nginx-ls
```

### Requirements

- Bash 4+
- `awk` (POSIX-compatible, works with macOS and GNU awk)
- `docker` (only if using `--container`)

## Usage

```
nginx-ls [OPTIONS]
```

### Sources (mutually exclusive)

| Flag | Description |
|------|-------------|
| *(default)* | Auto-detect local nginx or Docker container |
| `-c, --container NAME` | Read config from a specific Docker container |
| `-s, --stdin` | Read `nginx -T` output from stdin |

### Options

| Flag | Description |
|------|-------------|
| `--no-color` | Disable colored output |
| `--color` | Force colored output (even when piped) |
| `--no-header` | Skip the table header (useful for scripting) |
| `-a, --all` | Include catch-all/default server blocks |
| `-h, --help` | Show help |
| `-v, --version` | Show version |

## Examples

```bash
# Run on a server with nginx installed
nginx-ls

# Run against an nginx Docker container
nginx-ls -c my-nginx

# Pipe from a remote server over SSH
ssh my-server 'nginx -T 2>&1' | nginx-ls -s

# Pipe from a file
nginx-ls -s < nginx-config-dump.txt

# Include catch-all/default server blocks
nginx-ls --all

# Machine-readable (no header, no color)
nginx-ls --no-header --no-color
```

## How It Works

1. **Detects the config source** - local `nginx -T`, Docker container, or stdin
2. **Parses the merged config** - uses `awk` to walk through server blocks and extract `server_name`, `listen`, `root`, and `proxy_pass` directives
3. **Outputs a formatted table** - with color-coded columns (proxy, static, SSL indicators)

### What it extracts per server block

| Directive | Column | Notes |
|-----------|--------|-------|
| `server_name` | DOMAIN | Multiple names shown comma-separated, truncated at 50 chars |
| `listen` | LISTEN | Ports deduplicated, `ssl` flag shown |
| `root` | TARGET | Shown when no `proxy_pass` exists (TYPE = `static`) |
| `proxy_pass` | TARGET | Prefers the `location /` block; falls back to first found |

### Limitations

- Parses `http {}` blocks only (not `stream {}` for TCP/UDP)
- Shows `proxy_pass` values as-is (doesn't resolve `upstream` block names)
- Variable-based `proxy_pass` (e.g., `$backend`) shown literally
- Only the first/root `proxy_pass` per server block is shown

## License

MIT
