# Current Limitations

## Multi-File Application Code

Currently, Uzumibi requires that Ruby application code be written in a **single file** (`lib/app.rb` or `lib/consumer.rb`).

In typical Ruby development, it's common to split code across multiple files using `require` or `require_relative` for modularization. However, in Uzumibi, these file-loading features are not available due to mruby/edge's limitations. For now, the workaround is to write all code in a single file or concatenate files using a build script.

Multi-file support is a feature that will be prioritized in future Uzumibi development.

## Language Feature Limitations of mruby/edge

Since mruby/edge implements a subset of mruby, it has some limitations compared to CRuby (standard Ruby) or the full mruby:

- **Some standard libraries**: Libraries that depend on OS resources such as `File`, `IO`, and `Socket` are not available in the WebAssembly environment.
- **Regular expressions**: Regular expressions (`Regexp`) currently have only partial support.
- **`require` / `load`**: As mentioned above, loading external files is not supported.
- **Threads / Fiber**: Concurrency features are not available due to WebAssembly environment constraints.

Additionally, the Ruby features available may vary by platform in the future. We plan to document these as much as possible.

## Feature Differences Between Platforms

> **Note:** This section is updated periodically. Last updated: March 14, 2026.

While Uzumibi supports multiple platforms, external service integration features differ by platform:

| Feature | Cloudflare | Fastly | Spin | Cloud Run |
|---------|-----------|--------|------|-----------|
| Basic HTTP processing | o | o | o | o |
| External HTTP requests (Fetch) | o | - | - | - |
| Key-Value store | o (Durable Object) | - | - | - |
| Message queue | o (Queues) | - | - | - |
| Static asset serving | o | - | - | - |

Currently, Cloudflare Workers supports the most features. Support for other platforms is expected to expand progressively.
