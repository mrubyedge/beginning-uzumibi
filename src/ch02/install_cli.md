# Installing uzumibi-cli

To start developing with Uzumibi, you first need to install `uzumibi-cli` and its prerequisite tools.

## Prerequisites

The following tools must be installed beforehand.

### Rust Toolchain

The Rust compiler is required to build Uzumibi projects. Install it using `rustup`:

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

After installation, add the WASM target:

```bash
rustup target add wasm32-unknown-unknown
```

Verify the version:

```bash
$ rustc --version
rustc 1.93.1 (01f6ddf75 2026-02-11)
```

### Node.js and pnpm

For Cloudflare Workers, Node.js is required for development and deployment with Wrangler.

```bash
# Installing Node.js (example using a version manager)
# With fnm
fnm install --lts
# With nvm
nvm install --lts

# Installing pnpm
npm install -g pnpm
```

Verify the versions:

```bash
$ node --version
v25.6.1

$ pnpm --version
10.29.3
```

### Wrangler CLI

A CLI tool used for developing and deploying Cloudflare Workers. It will be automatically installed locally via `pnpm install` after project creation, but installing it globally can be convenient:

```bash
pnpm install -g wrangler
```

### clang / Build Tools

The C compiler `clang` is required for building the mruby compiler (`mruby-compiler2`).

For macOS:

```bash
xcode-select --install
```

For Linux:

```bash
# Ubuntu/Debian
sudo apt-get install clang build-essential

# Fedora
sudo dnf install clang
```

## Installing uzumibi-cli

`uzumibi-cli` is installed using Rust's package manager `cargo`:

```bash
cargo install 'uzumibi-cli@^0.6'
```

Once installation is complete, verify the version:

```bash
$ uzumibi --version
uzumibi-cli 0.6.1
```

You can view the help to see available commands:

```bash
$ uzumibi --help
```

With this, your development environment is ready.
