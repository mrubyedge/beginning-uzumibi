# Supported Platforms

Uzumibi supports multiple edge computing platforms. When creating a project with the `uzumibi new` command, you select the target platform using the `--template` option.

## Platform List

| Platform | Template Name | Status | WASM Target |
|----------|--------------|--------|-------------|
| Cloudflare Workers | `cloudflare` | Beta | `wasm32-unknown-unknown` |
| Fastly Compute@Edge | `fastly` | Experimental | `wasm32-wasip1` |
| Spin (Fermyon Cloud) | `spin` | Experimental | `wasm32-wasip1` |
| Google Cloud Run | `cloudrun` | Experimental | Native (container) |
| Service Worker | `serviceworker` | Experimental | `wasm32-unknown-unknown` |
| Web Worker | `webworker` | Experimental | `wasm32-unknown-unknown` |

## Cloudflare Workers

Cloudflare Workers is a platform for running serverless code on Cloudflare's global network. Uzumibi has the most advanced development for this platform, supporting the following features:

- Basic HTTP request/response processing
- Static asset serving (`public/` directory)
- External HTTP requests (`Uzumibi::Fetch`)
- Data persistence with Durable Objects (`Uzumibi::KV`)
- Asynchronous messaging with Cloudflare Queues (`Uzumibi::Queue`)

Development uses Node.js (pnpm), the Wrangler CLI, and the Rust toolchain.

## Fastly Compute@Edge

Fastly Compute@Edge is a platform for running WASM applications on Fastly's CDN edge nodes. It uses glue code written in Rust for request processing.

Development uses the Fastly CLI and the Rust toolchain.

## Spin (Fermyon Cloud)

Spin is a WebAssembly microservice framework developed by Fermyon. It can be deployed to Fermyon Cloud and run as a serverless application.

Development uses the Spin CLI and the Rust toolchain.

## Google Cloud Run (Experimental)

Google Cloud Run is a container-based serverless platform. In Uzumibi, the application is packaged as an HTTP server using Tokio + Hyper in a container, ready to run on Cloud Run.

Unlike other platforms, it is built as a **native binary** rather than Wasm. A Dockerfile is automatically included, so you can build a container from it.

Since the project is created in a form that can also run on localhost, development is possible as long as Docker is installed. Having the Rust toolchain available is also helpful.

## Service Worker / Web Worker (Experimental)

These are experimental templates for running Uzumibi applications on the client side using the browser's Service Worker API or Web Worker API.

The Rust toolchain is required for creating the Wasm.

## Disclaimer

> **Note:** The names and trademarks of each platform and software belong to their respective operating companies. This book cites these intellectual properties for introductory purposes.
