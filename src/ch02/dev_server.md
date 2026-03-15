# Starting the Dev Server

Once the application code is written, let's start the development server to verify it works.

## Launching the Dev Server

Start the development server with the following command:

```bash
pnpm run dev
```

This command internally executes the following steps in sequence:

1. **Rust build**: Runs the `cargo` command to compile `lib/app.rb` into mruby bytecode and then generate a Wasm module (`.wasm`).
2. **Wasm file copy**: Copies the built `.wasm` file to the `src/` directory.
3. **Wrangler dev startup**: Runs `wrangler dev` to start the local development server.

The first build may take several minutes as it needs to download and compile Rust crates. Subsequent builds will be faster due to incremental compilation.

When the build is complete, you'll see a message like this:

```
 ⛅️ wrangler 4.73.0
───────────────────

Your Worker has access to the following bindings:
Binding            Resource      Mode
env.ASSETS         Assets        local
...

⎔ Starting local server...
[wrangler:info] Ready on http://localhost:8787
```

## Verifying Operation

Once the development server is running, verify the operation using a browser or curl.

### Browser Verification

Access `http://localhost:8787` in your browser to see the contents of `public/index.html`. Clicking the "Call API" button will display the response from the `/api/hello` endpoint.

You can adjust the JavaScript in the HTML to call other endpoints and check their output as well.

### curl Verification

Let's call the API endpoints using curl:

```bash
# Root path (static assets)
$ curl http://localhost:8787/
<!DOCTYPE html>
<html lang="en">
...

# API endpoint
$ curl http://localhost:8787/api/hello
Hello from Uzumibi! Running on mruby/edge 3.2.0

# Dynamic parameters
$ curl http://localhost:8787/api/greet/World
{"message": "Hello, World!"}

# POST request
$ curl -X POST -d "test data" http://localhost:8787/api/echo
Received: test data
```

## Reflecting Code Changes

If you modify `lib/app.rb`, the current version of Uzumibi does not support automatic reloading. Since a Wasm rebuild is required, stop the development server (`Ctrl+C`) and run `pnpm run dev` again:

```bash
# Stop with Ctrl+C
# Edit code
# Restart
pnpm run dev
```

## Troubleshooting

### Build Error: `clang not found`

`clang` is required for building the mruby compiler. On macOS, install it with `xcode-select --install`; on Linux, use `apt install clang`.

### Build Error: `wasm32-unknown-unknown target not found`

The Wasm target may not have been added.

```bash
rustup target add wasm32-unknown-unknown
```

### Wrangler Authentication Error

If this is your first time using Wrangler, you may need to log in:

```bash
npx wrangler login
```

### Port Conflict

If the default port 8787 is in use by another process, Wrangler will automatically use a different port. Check the URL displayed in the console output.
