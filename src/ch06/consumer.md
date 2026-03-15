# Consumer Implementation

The consumer Worker receives messages from a Cloudflare Queue and processes them. In Uzumibi, you inherit from the `Uzumibi::Consumer` class and implement processing in the `on_receive` method.

## Project Structure

When the `queue` feature is enabled, the following file is generated instead of the standard `lib/app.rb`:

- `lib/consumer.rb` - Consumer Ruby code

Additionally, consumer-specific Wasm export functions (`uzumibi_initialize_message`, `uzumibi_start_message`) are added to `wasm-app/src/lib.rs`.

```
queue-consumer/
├── lib/
│   └── consumer.rb     # Queue consumer processing
├── src/
│   └── index.js        # JS glue code (handles both HTTP and Queue)
├── wasm-app/
│   ├── build.rs        # Compiles both app.rb and consumer.rb
│   └── src/
│       └── lib.rs      # Wasm (exports both HTTP and Queue processing)
├── wrangler.jsonc
└── package.json
```

The consumer Worker is dedicated to processing Queue messages.

## Verifying wrangler.jsonc

The consumer's `wrangler.jsonc` contains `queues.consumers` configuration. Double-check that the queue name matches the sender:

```jsonc
{
    "queues": {
        "producers": [
            {
                "binding": "UZUMIBI_QUEUE",
                "queue": "my-app-queue"
            }
        ],
        "consumers": [
            {
                "queue": "my-app-queue",
                "max_batch_size": 10,
                "max_batch_timeout": 5
            }
        ]
    }
}
```

## Implementing the Consumer

Edit `lib/consumer.rb` to implement the message processing logic:

```ruby
class Consumer < Uzumibi::Consumer
  # @rbs message: Uzumibi::Message
  def on_receive(message)
    debug_console("[Consumer] Received message: id=#{message.id}, body=#{message.body}, attempts=#{message.attempts}")

    # Process the message
    body = message.body
    debug_console("[Consumer] Processing: #{body}")

    # Acknowledge after successful processing
    if message.attempts > 3
      # Give up and ack after more than 3 retries
      debug_console("[Consumer] Giving up after #{message.attempts} attempts, acknowledging message #{message.id}")
      message.ack!
    else
      # Normal processing
      begin
        process_message(body)
        debug_console("[Consumer] Successfully processed message #{message.id}")
        message.ack!
      rescue => e
        debug_console("[Consumer] Error processing message: retrying in 5 seconds")
        message.retry(delay_seconds: 5)
      end
    end
  end

  def process_message(body)
    # Actual message processing logic
    debug_console("[Consumer] Message content: #{body}")
    # Write specific processing here
    # e.g., calling external APIs, saving data, etc.
  end
end

$CONSUMER = Consumer.new
```

### Code Walkthrough

**Inheriting `Uzumibi::Consumer`**

```ruby
class Consumer < Uzumibi::Consumer
  def on_receive(message)
    # ...
  end
end
```

Inherit from `Uzumibi::Consumer` and override the `on_receive` method. This method is called each time a message is received from the queue.

**Message Attributes**

The `message` object (`Uzumibi::Message`) passed to `on_receive` provides the following information:

```ruby
message.id        # Message ID (string)
message.body      # Message body (string)
message.attempts  # Delivery attempt count (integer)
message.timestamp # Send timestamp (ISO 8601 format string)
```

**Acknowledgment (ack!)**

```ruby
message.ack!
```

Call `ack!` on a message once processing is complete. Messages that have been acknowledged are removed from the queue.

**Retry**

```ruby
message.retry(delay_seconds: 5)
```

If processing fails, call `retry` to redeliver the message. Specify the delay time until redelivery in seconds with `delay_seconds`. The next time `on_receive` is called, `message.attempts` will be incremented.

**Global Variable `$CONSUMER`**

```ruby
$CONSUMER = Consumer.new
```

Assign the consumer instance to the global variable `$CONSUMER`. This is referenced from the Rust code on the Wasm side, so it must always be set.

## Deploying for Verification

> **Note:** With the current Wrangler, Queue communication across multiple projects cannot be verified in the localhost development environment.

Deploy queue-consumer for verification.

First, perform the build:

```bash
pnpm install
pnpm run dev
# The HTTP part is empty, so it starts but behavior can't be verified.
# This is just to build the Wasm, so stop it afterwards.
```

## Verifying Operation

### Testing

To check the received logs, display logs in the terminal with the following command:

Terminal (queue-consumer):
```bash
$ cd queue-consumer
$ npx wrangler tail

 ⛅️ wrangler 4.73.0
───────────────────
Successfully created tail, expires at 2026-03-14T19:19:39Z
```

In a separate terminal, issue the following curl command to send a message:

```bash
curl -X POST -d "Test message from publisher" \
    http://queue-publisher.<ID>.workers.dev/api/send
{"message":"Test message from publisher","status":"accepted"}
```

Message processing logs will appear in the queue-consumer console:

```
Queue my-app-queue (1 message) - Ok @ 2026/3/14 22:22:05
  (log) [debug]: [Consumer] Received message: id=25f4ef4ca8b2fdb7055cc5b9XXXXXXXX, body=Test message from publisher, attempts=1
  (log) [debug]: [Consumer] Processing: Test message from publisher
  (log) [debug]: [Consumer] Message content: Test message from publisher
  (log) [debug]: [Consumer] Successfully processed message 25f4ef4ca8b2fdb7055cc5b9XXXXXXXX
```

Processing status can also be checked from the Cloudflare dashboard under "Workers & Pages" > target Worker > "Logs".

## Chapter Summary

Queues enable easy asynchronous processing with Cloudflare Workers + Uzumibi.

When using Queues, the architecture is somewhat special. Please note that with Uzumibi, two Workers are required.
