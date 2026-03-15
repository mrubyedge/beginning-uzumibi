# Uzumibi and External Services

This chapter provides an overview of the external services supported by Uzumibi on Cloudflare Workers.

## External Services Available on Uzumibi with Cloudflare Workers

On Cloudflare Workers, you can access external Web APIs through JavaScript's `fetch()` function, as well as various services provided by Cloudflare.

Uzumibi provides APIs for using some of these services from Ruby.

To use external service integration, specify the `--features enable-external` option when creating a project:

```bash
uzumibi new --template cloudflare --features enable-external my-app
```

Currently, Uzumibi on Cloudflare Workers supports the following three external services:

## Fetch

`Uzumibi::Fetch` is an API for making external HTTP requests from within a Worker. Use it when calling external REST APIs or web services.

```ruby
# Basic usage
response = Uzumibi::Fetch.fetch("https://api.example.com/data")
# response.status_code => 200
# response.body => Response body
# response.headers => Response headers (Hash)
```

The `Uzumibi::Fetch.fetch` method takes the following four arguments:

| Argument | Type | Description |
|----------|------|-------------|
| `url` | String | Request URL (required) |
| `method` | String | HTTP method (defaults to `"GET"`) |
| `body` | String | Request body (defaults to empty string) |
| `headers` | Hash[String, String] | Request headers (defaults to none) |

The return value is a `Uzumibi::Response` object with `status_code`, `headers`, and `body`.

Internally, it executes asynchronous HTTP requests through JavaScript's `fetch()` API.

> **Note:** Functions like `fetch()` are asynchronous at the JavaScript level. To call asynchronous operations through Wasm, Uzumibi uses the `asyncify` feature from [binaryen](https://github.com/WebAssembly/binaryen) and the `asyncify-wasm` JavaScript library. Due to these libraries, the Wasm binary is slightly larger when external service integration is enabled.

## Durable Object

`Uzumibi::KV` is a Key-Value store that uses Cloudflare Durable Objects. It allows you to persistently store data across requests.

```ruby
# Save a value
Uzumibi::KV.set("key", "value")

# Get a value
value = Uzumibi::KV.get("key")
# => "value"

# Non-existent key
value = Uzumibi::KV.get("unknown")
# => nil
```

Durable Objects are Cloudflare Workers' stateful storage feature. In Uzumibi, a Durable Object class named `UzumibiKVObject` is automatically configured, and data is persisted using SQLite-based storage.

| Method | Arguments | Return Value | Description |
|--------|-----------|-------------|-------------|
| `Uzumibi::KV.get(key)` | key: String | String or nil | Get the value for a key |
| `Uzumibi::KV.set(key, value)` | key: String, value: String | true | Save a value for a key |

## Queue

`Uzumibi::Queue` is an API for sending messages to Cloudflare Queues. It can be used for asynchronous background processing. When using Queues, you need to create a Queue in advance through the Cloudflare dashboard or similar.

```ruby
# Send a message
Uzumibi::Queue.send("UZUMIBI_QUEUE", "Hello from queue!")
```

| Method | Arguments | Return Value | Description |
|--------|-----------|-------------|-------------|
| `Uzumibi::Queue.send(queue_name, message)` | queue_name: String, message: String | true | Send a message to the specified queue |

`queue_name` is the name of the queue binding defined in `wrangler.jsonc`. The implementation of the Queue receiver (Consumer) is explained in detail in Chapter 6.

## Internal Behavior of External Service Features

Projects with external service integration enabled differ from standard projects in the following ways:

1. **Wasm module**: The `enable-external` feature is added to `wasm-app/Cargo.toml`, incorporating Fetch/KV/Queue wrapper functions on the Rust side.
2. **JavaScript glue code**: `src/index.js` is replaced with a version that uses the `asyncify-wasm` library, enabling asynchronous operations (`await`) inside Wasm.
3. **wrangler.jsonc**: Durable Object bindings and migration settings are added. Queue enablement needs to be configured manually.

---

In the following chapters, we'll introduce examples that actually use the external services.
