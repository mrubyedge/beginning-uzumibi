# Queue Architecture and Setup

When building a Uzumibi application using Cloudflare Queues, you configure two Workers: a sender (Publisher) and a receiver (Consumer). This section explains the overall architecture and setup process.

## Architecture

```
Client
    ↓ HTTP request
[Publisher Worker] (generated with enable-external feature)
    ↓ Uzumibi::Queue.send
[Cloudflare Queue]
    ↓ Message delivery
[Consumer Worker] (generated with queue feature)
    ↓ on_receive(message)
    Message processing → ack! or retry
```

The sender and receiver operate as separate Workers, both bound to the same queue. The sender sends messages through the `queues.producers` binding, and the receiver receives messages through the `queues.consumers` binding.

## Creating a Cloudflare Queue

First, you need to create a queue in Cloudflare Queues. Create one from the Cloudflare dashboard or using the Wrangler CLI:

```bash
$ npx wrangler queues create my-app-queue
...
 ⛅️ wrangler 4.73.0
───────────────────
🌀 Creating queue 'my-app-queue'
✅ Created queue 'my-app-queue'
```

Verify that the queue was created:

```bash
$ npx wrangler queues list
┌──────────────────────────────────┬───────────────┬─────────────────────────────┬─────────────────────────────┬───────────┬───────────┐
│ id                               │ name          │ created_on                  │ modified_on                 │ producers │ consumers │
├──────────────────────────────────┼───────────────┼─────────────────────────────┼─────────────────────────────┼───────────┼───────────┤
│ 7941fbb9762d4a02b1a1c644XXXXXXXX │ my-app-queue  │ 2026-03-14T12:48:00.918834Z │ 2026-03-14T12:48:00.918834Z │ 0         │ 0         │
└──────────────────────────────────┴───────────────┴─────────────────────────────┴─────────────────────────────┴───────────┴───────────┘
```

## Project Structure

Two projects are needed for Queue communication:

1. **Sender project**: Created with the `enable-external` feature, sends messages using `Uzumibi::Queue.send`.
2. **Receiver project**: Created with the `queue` feature, receives and processes messages using `Uzumibi::Consumer`.

```bash
# Sender
uzumibi new --template cloudflare --features enable-external queue-publisher
# Receiver
uzumibi new --template cloudflare --features queue queue-consumer
```

> **Note:** When the `queue` feature is enabled, the `enable-external` feature is also automatically enabled in the generated project.

## wrangler.jsonc Configuration

### Sender Configuration

In the sender's `wrangler.jsonc`, configure the queue binding in `queues.producers`. The default template has this commented out, so uncomment it to enable:

```jsonc
{
    "queues": {
        "producers": [
            {
                "binding": "UZUMIBI_QUEUE",
                "queue": "my-app-queue"
            }
        ]
    }
}
```

`binding` is the name used to reference the queue in code, corresponding to the first argument of `Uzumibi::Queue.send`. `queue` specifies the actual queue name created in Cloudflare Queues.

### Receiver Configuration

In the receiver's `wrangler.jsonc`, the `queues.consumers` queue binding is automatically configured from the project name. For this example, manually change it to the correct name:

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

Configure the following values as needed:

| Setting | Description |
|---------|-------------|
| `queue` | Name of the queue to receive from |
| `max_batch_size` | Maximum number of messages to receive at once (default: 10) |
| `max_batch_timeout` | Maximum wait time in seconds for a batch to fill (default: 5) |
