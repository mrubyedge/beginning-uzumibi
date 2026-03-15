# Saving Memos with Durable Object

In this chapter, we'll create a memo application that stores data persistently using `Uzumibi::KV` (based on Cloudflare Durable Objects).

## Architecture Overview

Cloudflare Durable Objects is a service for managing stateful objects on Cloudflare Workers. While regular Workers are stateless (they don't retain state between requests), Durable Objects enable persistent data read/write operations.

In Uzumibi version 0.6.x, the `Uzumibi::KV` API wraps Durable Objects as a simple Key-Value store.

```
Client → Cloudflare Worker (Uzumibi)
            ↓ Uzumibi::KV.get/set
         UzumibiKVObject (Durable Object)
            ↓
         SQLite storage (persisted)
```

`UzumibiKVObject` is a Durable Object class defined in the JavaScript glue code. Internally, it uses the Durable Object's `ctx.storage` API (SQLite-based) to store data.

## Creating the Project

Create a project with external service integration enabled:

```bash
uzumibi new \
  --template cloudflare \
  --features enable-external memo-app
cd memo-app
pnpm install
```

Check the generated `wrangler.jsonc` to see the automatically configured Durable Object bindings:

```jsonc
{
    "durable_objects": {
        "bindings": [
            {
                "name": "UZUMIBI_KV_DATA",
                "class_name": "UzumibiKVObject"
            }
        ]
    },
    "migrations": [
        {
            "tag": "v1",
            "new_sqlite_classes": [
                "UzumibiKVObject"
            ]
        }
    ]
}
```

## Implementation

Edit `lib/app.rb` to create a simple memo save/retrieve API:

```ruby
class App < Uzumibi::Router
  get "/" do |req, res|
    fetch_assets
  end

  # Get a memo
  get "/api/memo/:key" do |req, res|
    key = req.params[:key]
    value = Uzumibi::KV.get(key)

    if value
      res.status_code = 200
      res.headers = {
        "content-type" => "application/json",
      }
      res.body = JSON.generate({
        key: key,
        value: value
      })
    else
      res.status_code = 404
      res.headers = {
        "content-type" => "application/json",
      }
      res.body = JSON.generate({
        error: "not found",
        key: key
      })
    end
    res
  end

  # Save a memo
  post "/api/memo/:key" do |req, res|
    key = req.params[:key]
    value = req.body

    debug_console("Saving memo: #{key} = #{value}")
    Uzumibi::KV.set(key, value)

    res.status_code = 201
    res.headers = {
      "content-type" => "application/json",
    }
    res.body = JSON.generate({
      key: key,
      value: value,
      status: "saved"
    })
    res
  end
end

$APP = App.new
```

### Code Walkthrough

**Getting a Value**

```ruby
value = Uzumibi::KV.get(key)
```

`Uzumibi::KV.get` returns the value for the specified key as a string. If the key doesn't exist, it returns `nil`.

**Saving a Value**

```ruby
Uzumibi::KV.set(key, value)
```

`Uzumibi::KV.set` saves a key-value pair. Calling `set` again with the same key overwrites the value. Both keys and values are string types.

## Verifying Operation

Start the development server:

```bash
pnpm run dev
```

Test saving and retrieving memos with curl:

```bash
# Save a memo
$ curl -X POST -d "Shopping list: milk, bread, eggs" \
  http://localhost:8787/api/memo/shopping
{"key": "shopping", "value": "Shopping list: milk, bread, eggs", "status": "saved"}

# Get a memo
$ curl http://localhost:8787/api/memo/shopping
{"key": "shopping", "value": "Shopping list: milk, bread, eggs"}

# Non-existent key
$ curl http://localhost:8787/api/memo/unknown
{"error": "not found", "key": "unknown"}

# Update a memo
$ curl -X POST -d "Shopping list: milk, bread, eggs, butter" \
  http://localhost:8787/api/memo/shopping
{"key": "shopping", "value": "Shopping list: milk, bread, eggs, butter", "status": "saved"}
```

Data stored in the Durable Object persists across Worker restarts and deployments.

## Notes

- Both keys and values in `Uzumibi::KV` are string types. If you want to store numbers or objects, convert them to JSON strings or similar before saving.
- During local development (`wrangler dev`), Durable Object data is saved in local storage within the project's `.wrangler/` directory.
- When deployed to production, data is stored on Cloudflare's global network.
