# Calling External Public APIs with Fetch

In this chapter, we'll create an application that calls external Web APIs using `Uzumibi::Fetch`.

## Creating the Project

Create a new project with external service integration enabled:

```bash
uzumibi new \
  --template cloudflare \
  --features enable-external \
  fetch-example
cd fetch-example
pnpm install
```

## Selecting an API to Use

In this example, we'll use the freely available [JSONPlaceholder](https://jsonplaceholder.typicode.com/) API. This is a test API that returns dummy TODO data, user data, and more.

We'll primarily use the following endpoints:

- `GET https://jsonplaceholder.typicode.com/todos/1` - Get a TODO by ID
- `GET https://jsonplaceholder.typicode.com/todos?_limit=5` - Get a list of TODOs (with a limit)

## Calling the API

Edit `lib/app.rb` as follows:

```ruby
class App < Uzumibi::Router
  get "/" do |req, res|
    fetch_assets
  end

  # Get a single TODO from the external API
  get "/api/todo/:id" do |req, res|
    id = req.params[:id]
    api_url = "https://jsonplaceholder.typicode.com/todos/#{id}"

    debug_console("Fetching: #{api_url}")
    api_response = Uzumibi::Fetch.fetch(api_url)

    res.status_code = api_response.status_code
    res.headers = {
      "content-type" => "application/json",
    }
    res.body = api_response.body
    res
  end

  # Get a list of TODOs from the external API
  get "/api/todos" do |req, res|
    api_url = "https://jsonplaceholder.typicode.com/todos?_limit=5"

    debug_console("Fetching: #{api_url}")
    api_response = Uzumibi::Fetch.fetch(api_url)

    res.status_code = api_response.status_code
    res.headers = {
      "content-type" => "application/json",
    }
    res.body = api_response.body
    res
  end

  # Example of forwarding a POST request
  post "/api/todo" do |req, res|
    api_url = "https://jsonplaceholder.typicode.com/todos"

    debug_console("Posting to: #{api_url}")
    api_response = Uzumibi::Fetch.fetch(
      api_url, "POST",
      JSON.generate(req.body),
      {"content-type" => "application/json"}
    )

    res.status_code = api_response.status_code
    res.headers = {
      "content-type" => "application/json",
    }
    res.body = api_response.body
    res
  end
end

$APP = App.new
```

### Code Walkthrough

**Forwarding GET Requests**

```ruby
api_response = Uzumibi::Fetch.fetch(api_url)
```

Simply passing a URL to `Uzumibi::Fetch.fetch` executes a GET request. The return value `api_response` is a `Uzumibi::Response` object that holds the response from the external API.

**Forwarding POST Requests**

```ruby
api_response = Uzumibi::Fetch.fetch(api_url, "POST", req.body)
```

By specifying an HTTP method as the second argument and a request body as the third argument, you can send POST and other types of requests.

Specify header information such as authentication as a Hash in the fourth argument.

**Debug Output**

```ruby
debug_console("Fetching: #{api_url}")
```

The `debug_console` method outputs debug messages to the Wrangler console (terminal). It is not displayed in the browser.

## Verifying Operation

Start the development server:

```bash
pnpm run dev
```

Verify operation with curl:

```bash
# Get a single TODO
$ curl http://localhost:8787/api/todo/1
{
  "userId": 1,
  "id": 1,
  "title": "delectus aut autem",
  "completed": false
}

# Get a list of TODOs
$ curl http://localhost:8787/api/todos
[
  {
    "userId": 1,
    "id": 1,
    "title": "delectus aut autem",
    "completed": false
  },
  {
    "userId": 1,
    "id": 2,
    "title": "quis ut nam facilis et officia qui",
    "completed": false
  },...
]

# POST request
$ curl -X POST -H "Content-Type: application/json" \
    -d '{"title":"New Todo","completed":false,"userId":1}' \
    http://localhost:8787/api/todo
{
  "completed": false,
  "title": "New Todo",
  "userId": 1,
  "id": 201
}
```

Debug messages like the following will appear in the Wrangler console:

```
[debug]: Fetching: https://jsonplaceholder.typicode.com/todos/1
```

We've implemented a sample application that calls external APIs from Uzumibi.

## Chapter Summary

With `Uzumibi::Fetch`, you can easily integrate with external microservices and third-party APIs.
