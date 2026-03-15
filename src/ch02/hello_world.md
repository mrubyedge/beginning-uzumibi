# Implementing a Hello World Application

Now that the project is created, let's implement an actual application. Uzumibi allows you to work with both Ruby code (`lib/app.rb`) and a static frontend (`public/` directory).

## Frontend Implementation

First, let's place a static HTML file in the `public/` directory. Create `public/index.html`:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Hello Uzumibi</title>
    <style>
        body {
            font-family: sans-serif;
            max-width: 600px;
            margin: 50px auto;
            padding: 0 20px;
        }
        #result {
            margin-top: 20px;
            padding: 15px;
            background: #f0f0f0;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <h1>Hello Uzumibi!</h1>
    <p>Building Cloudflare Workers with Ruby</p>

    <button id="greetBtn">Call API</button>
    <div id="result"></div>

    <script>
        document.getElementById('greetBtn').addEventListener('click', async () => {
            const res = await fetch('/api/hello');
            const text = await res.text();
            document.getElementById('result').textContent = text;
        });
    </script>
</body>
</html>
```

Files placed in the `public/` directory are automatically served by Cloudflare Workers' static asset feature.

## API Implementation

Next, edit `lib/app.rb` to implement the API endpoints. Replace the default generated code with the following:

```ruby
class App < Uzumibi::Router
  # Root path delegates to static assets (public/index.html)
  get "/" do |req, res|
    fetch_assets
  end

  # Simple API endpoint
  get "/api/hello" do |req, res|
    res.status_code = 200
    res.headers = {
      "content-type" => "text/plain",
      "x-powered-by" => "#{RUBY_ENGINE} #{RUBY_VERSION}"
    }
    res.body = "Hello from Uzumibi! Running on #{RUBY_ENGINE} #{RUBY_VERSION}\n"
    res
  end

  # Greeting API with dynamic parameters
  get "/api/greet/:name" do |req, res|
    name = req.params[:name]
    res.status_code = 200
    res.headers = {
      "content-type" => "application/json",
    }
    res.body = JSON.generate({ message: "Hello, #{name}!" })
    res
  end

  # POST request handling
  post "/api/echo" do |req, res|
    res.status_code = 200
    res.headers = {
      "content-type" => "text/plain",
    }
    res.body = "Received: #{req.body.inspect}\n"
    res
  end
end

$APP = App.new
```

## Code Walkthrough

### Routing

Define routes using class methods like `get` and `post` within a class that inherits from `Uzumibi::Router`:

```ruby
class App < Uzumibi::Router
  get "/path" do |req, res|
    # Handler logic
  end
end
```

Each route handler block receives two arguments: `req` (request) and `res` (response).

### Serving Static Assets

```ruby
get "/" do |req, res|
  fetch_assets
end
```

Calling the `fetch_assets` method delegates the request to the static assets in the `public/` directory. This causes `public/index.html` to be returned for the `/` path.

Note that if you define a handler for `"/file"` and a request comes in for `/file`, `public/file.html` will be returned if it exists. Be aware that the file extension is automatically appended.

### Building a Response

Set `status_code`, `headers`, and `body` on the response object, then return `res`:

```ruby
get "/api/hello" do |req, res|
  res.status_code = 200
  res.headers = {
    "content-type" => "text/plain",
  }
  res.body = "Hello!\n"
  res  # Always return res
end
```

### Dynamic Parameters

Including `:parameter_name` in the URL path captures that path segment as a dynamic parameter. Access it through the `req.params` Hash:

```ruby
get "/api/greet/:name" do |req, res|
  name = req.params[:name]
  # ...
end
```

### Request Body

The body of POST requests and similar can be accessed synchronously via `req.body`. If the request's `content-type` is form data (`application/x-www-form-urlencoded`) or JSON data (`application/json`), it is automatically parsed, and you can access the data through `req.params` as well.

### The `$APP` Global Variable

```ruby
$APP = App.new
```

In Uzumibi `0.6.x`, the convention is to assign the application instance to the global variable `$APP` at the end. This variable is referenced from the Rust code on the Wasm side, so it must always be set.
