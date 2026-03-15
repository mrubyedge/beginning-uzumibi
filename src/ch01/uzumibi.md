# What is Uzumibi

Uzumibi is a lightweight framework for building web applications in Ruby on edge computing platforms, built on top of mruby/edge.

The name "Uzumibi" is inspired by the popular edge framework [Hono](https://hono.dev/). Hono + Embedded (埋まっている / "buried") = Uzumibi (うずみび / "buried embers") 😅

## Overview of Uzumibi

Uzumibi is a web application framework that allows you to define routing using a Sinatra-like DSL. You can write applications that handle HTTP requests with Ruby code like the following:

```ruby
class App < Uzumibi::Router
  get "/" do |req, res|
    res.status_code = 200
    res.headers = {
      "content-type" => "text/plain",
      "x-powered-by" => "#{RUBY_ENGINE} #{RUBY_VERSION}"
    }
    res.body = "It works!\n"
    res
  end

  get "/greet/to/:name" do |req, res|
    res.status_code = 200
    res.headers = {
      "content-type" => "text/plain",
    }
    res.body = "Hello, #{req.params[:name]}!!\n"
    res
  end
end

$APP = App.new
```

## How It Works

Uzumibi applications operate in a two-layer architecture:

1. **Wasm layer (Rust + mruby/edge)**: Ruby code is compiled into mruby bytecode and embedded in a Wasm module. Request processing is handled by Ruby code running on the mruby/edge VM.
2. **Platform layer (JavaScript / Rust)**: Glue code that bridges the edge platform's native APIs with the Wasm module. It handles binary serialization of HTTP requests/responses.

The request flow looks like this:

```
Client
    → Edge platform
    → Glue code (JS/Rust) serializes request to binary
    → Ruby code executes on mruby/edge VM inside WASM module
    → Response is deserialized from binary
    → Response returned to client
```

## Routing

Uzumibi's router supports the following HTTP methods:

- `get`
- `post`
- `put`
- `delete`
- `head`
- `options`

URL paths can include dynamic parameters (`:name`) and wildcards (`*`). Parameters are accessible via `req.params`, which is a Hash.

```ruby
# Dynamic parameter example
get "/users/:id" do |req, res|
  user_id = req.params[:id]
  # ...
end
```

In this book, we refer to the Ruby blocks defined with `get`, `post`, etc. as "route handlers".

## Request Object

The `req` object (`Uzumibi::Request`) passed to the route handler block holds the following information:

- `req.method` - HTTP method (GET, POST, etc.)
- `req.path` - Request path
- `req.query` - Query string
- `req.headers` - Request headers (Hash)
- `req.body` - Request body
- `req.params` - A Hash that integrates URL parameters, query parameters, and form data

## Response Object

The `res` object (`Uzumibi::Response`) is used to construct the response by setting the following properties:

- `res.status_code` - HTTP status code (integer)
- `res.headers` - Response headers (Hash)
- `res.body` - Response body (string)

The route handler block must always return the `res` object.

## External Service Integration

Uzumibi also supports access to external services provided by edge platforms. On Cloudflare Workers, the following are available:

- **`Uzumibi::Fetch`** - Call external HTTP APIs
- **`Uzumibi::KV`** - Key-Value store using Durable Objects
- **`Uzumibi::Queue`** - Asynchronous messaging with Cloudflare Queues

These features are explained in detail in later chapters.

> **Note:** As of March 14, 2026, external services are not available on platforms other than Cloudflare Workers. Support for other platforms will be added progressively. Pull requests are always welcome!
