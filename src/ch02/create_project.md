# Creating a Project

Use the `uzumibi new` command to create a project for Cloudflare Workers.

## Generating the Project

Create a project named `hello-uzumibi` with the following command:

```bash
uzumibi new --template cloudflare hello-uzumibi
```

Running this command will create a project directory and generate the necessary files:

```
Creating project 'hello-uzumibi'...
  generate  hello-uzumibi/.gitignore
  generate  hello-uzumibi/Cargo.toml
  generate  hello-uzumibi/package.json
  generate  hello-uzumibi/vitest.config.js
  generate  hello-uzumibi/wrangler.jsonc
  generate  hello-uzumibi/lib/app.rb
  generate  hello-uzumibi/public/assets/index.html
  generate  hello-uzumibi/src/index.js
  generate  hello-uzumibi/wasm-app/Cargo.toml
  generate  hello-uzumibi/wasm-app/build.rs
  generate  hello-uzumibi/wasm-app/src/lib.rs

✓ Successfully created project from template 'cloudflare'
  Run 'cd hello-uzumibi' to get started!
```

## Project Directory Structure

The generated project has the following structure:

```
hello-uzumibi/
├── Cargo.toml          # Rust workspace configuration
├── package.json        # Node.js dependencies and scripts
├── wrangler.jsonc      # Wrangler (Cloudflare Workers CLI) configuration
├── lib/
│   └── app.rb          # Ruby application code (main)
├── public/
│   └── assets/...      # Static assets (HTML, CSS, images, etc.)
├── src/
│   └── index.js        # JavaScript glue code (entry point)
└── wasm-app/
    ├── Cargo.toml      # WASM crate configuration
    ├── build.rs        # Build script (compiles Ruby code)
    ├── src/
    │   └── lib.rs      # Rust code for the WASM module
    └── .cargo/
        └── config.toml # Cargo target settings (may not exist)
```

## Role of Each File

### `lib/app.rb`

**This is the main file that developers edit.** You write Uzumibi's routing and request handling logic in Ruby here.

### `src/index.js`

The entry point for Cloudflare Workers. It receives HTTP requests, serializes them to binary format and passes them to the WASM module, then deserializes the response and returns it to the client. Normally, you don't need to edit this file.

### `wasm-app/`

A Rust crate for compiling Ruby code into mruby bytecode and packaging it as a WASM module. Normally, you don't need to edit these files.

- `build.rs` contains configuration to compile `lib/app.rb` into mruby bytecode (`.mrb`) at build time.
- `src/lib.rs` handles mruby/edge VM initialization and export function definitions.

### `wrangler.jsonc`

The Cloudflare Workers configuration file. It contains the application name, static asset settings, and more.

```jsonc
{
    "name": "hello-uzumibi",
    "main": "src/index.js",
    "compatibility_date": "2025-12-30",
    "assets": {
        "directory": "./public",
        "binding": "ASSETS"
    }
}
```

### `package.json`

Defines npm/pnpm scripts and dependencies.

```json
{
    "scripts": {
        "deploy": "wrangler deploy",
        "dev": "cargo build --package hello-uzumibi --target wasm32-unknown-unknown --release && cp -v -f target/wasm32-unknown-unknown/release/hello_uzumibi.wasm src/ && wrangler dev",
        "start": "wrangler dev",
        "test": "vitest"
    }
}
```

The `dev` script performs both the Rust build (WASM compilation) and Wrangler dev server startup in one step.

## Installing Dependencies

Navigate to the project directory and install the Node.js dependencies:

```bash
cd hello-uzumibi
pnpm install
```

This installs development tools including Wrangler locally within the project.
