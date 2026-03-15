# Queue Use Cases

## Publisher

A Publisher (producer) is responsible for sending messages to a queue. In Uzumibi, you use the `Uzumibi::Queue.send` method to send messages.

Typical use cases include:

- **Asynchronous background processing**: Return a response to the user immediately while delegating time-consuming tasks (email sending, image conversion, data aggregation, etc.) to the queue.
- **Event notification**: Notify another Worker of events that occurred in one Worker (user registration, order completion, etc.) to trigger subsequent processing.
- **Rate limit mitigation**: Put requests to external APIs into a queue first, then process them sequentially on the consumer side in a controlled manner.

```ruby
# Example: Send a welcome email asynchronously after user registration
post "/api/users" do |req, res|
  # User registration processing (completes immediately)
  # ...

  # Delegate email sending to the queue (asynchronous)
  Uzumibi::Queue.send("UZUMIBI_QUEUE", "welcome_email:#{user_email}")

  res.status_code = 201
  res.body = JSON.generate({status: "registered"})
  res
end
```

## Consumer

A Consumer is responsible for receiving and processing messages from the queue.

> **Note:** In a standard Cloudflare project, the web application that sends to a Cloudflare Queue and the process that receives messages from the queue can be defined in the same Worker file. However, in Uzumibi 0.6.x, you **need to generate a separate consumer-only application**, so please keep this in mind.

In the Ruby code of the consumer application, you prepare a class that inherits from `Uzumibi::Consumer` and override the `on_receive` method to implement the processing logic.

Consumers have the following characteristics:

- **Batch processing**: Messages are received in batches (up to 10 messages, with a maximum wait time of 5 seconds).
- **Retry functionality**: Failed messages can be retried after a specified delay.
- **Acknowledgment**: Successfully processed messages are acknowledged with `ack!` and removed from the queue.

```ruby
class Consumer < Uzumibi::Consumer
  def on_receive(message)
    debug_console("Processing: #{message.body}")

    # Acknowledge after successful processing
    message.ack!
  end
end
```

The Message object has the following attributes:

| Attribute | Type | Description |
|-----------|------|-------------|
| `message.id` | String | Unique ID of the message |
| `message.timestamp` | String | Timestamp when the message was sent (ISO 8601 format) |
| `message.body` | String | Message body |
| `message.attempts` | Integer | Number of delivery attempts so far |

The Message object also has the following methods:

| Method | Description |
|--------|-------------|
| `message.ack!` | Notify that message processing is complete and remove it from the queue |
| `message.retry(delay_seconds: N)` | Redeliver the message after N seconds |
