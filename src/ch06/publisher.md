# Publisher Implementation

The publisher Worker (queue-publisher) receives HTTP requests and sends messages to a Cloudflare Queue.

## Application Implementation

Edit `lib/app.rb` as follows:

```ruby
class App < Uzumibi::Router
  # Message sending API
  post "/api/send" do |req, res|
    message = req.body

    if message == "" || message == nil
      res.status_code = 400
      res.headers = {
        "content-type" => "application/json",
      }
      res.body = JSON.generate({ error: "message body is required" })
    else
      debug_console("Sending message to queue: #{message}")
      Uzumibi::Queue.send("UZUMIBI_QUEUE", message)

      res.status_code = 202
      res.headers = {
        "content-type" => "application/json",
      }
      res.body = JSON.generate({ status: "accepted", message: message })
    end
    res
  end

  # Example of enqueuing a task
  post "/api/tasks/:task_type" do |req, res|
    task_type = req.params[:task_type]
    payload = req.body

    # Send task type and payload in JSON format
    queue_message = JSON.generate({ type: task_type, payload: payload })
    debug_console("Enqueuing task: #{queue_message}")
    Uzumibi::Queue.send("UZUMIBI_QUEUE", queue_message)

    res.status_code = 202
    res.headers = {
      "content-type" => "application/json",
    }
    res.body = JSON.generate({ status: "queued", task_type: task_type })
    res
  end
end

$APP = App.new
```

### Code Walkthrough

**Sending a Message**

```ruby
Uzumibi::Queue.send("UZUMIBI_QUEUE", message)
```

The first argument of `Uzumibi::Queue.send` specifies the `binding` name configured in `queues.producers` in `wrangler.jsonc`. The second argument is the message body (string).

**HTTP Status Code 202**

Since enqueuing a message means starting asynchronous processing, we return HTTP status code `202 Accepted`. The actual message processing is performed by the consumer Worker.

## Verifying Operation

Start the development server and test message sending:

```bash
pnpm run dev
```

```bash
# Send a message
$ curl -X POST -d "Hello, Queue\!" http://localhost:8787/api/send
{"status": "accepted", "message": "Hello, Queue!"}

# Enqueue a task
$ curl -X POST -d "user@example.com" \
    http://localhost:8787/api/tasks/send_email
{"status": "queued", "task_type": "send_email"}
```

During local development, queue messages are actually sent to a dummy server on localhost. Debug messages will be displayed in the Wrangler console:

```
[debug]: Sending message to queue: Hello, Queue!
[debug]: Enqueuing task: {"type": "send_email", "payload": "user@example.com"}
```

## Deploy for Integration Testing

After basic verification, deploy to the remote Cloudflare Worker with the following command:

```bash
pnpm run deploy
```

After deployment, perform a quick operational check:

```bash
curl -X POST -d "Hello, Production Queue\!" \
    http://queue-publisher.<ID>.workers.dev/api/send
{"message":"Hello, Production Queue!","status":"accepted"}
```
