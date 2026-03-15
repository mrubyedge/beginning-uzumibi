# What is mruby/edge

mruby/edge is a lightweight mruby implementation specialized for WebAssembly (Wasm) environments. It leverages the bytecode specification of mruby, a lightweight Ruby VM, and was developed with the goal of running efficiently in edge computing and serverless environments.

## What is mruby

[mruby](https://mruby.org/) is a lightweight implementation of the Ruby language designed to run in embedded environments and resource-constrained systems. It is primarily developed under the leadership of Yukihiro Matsumoto, the creator of Ruby. Compared to CRuby (the standard Ruby implementation), mruby has a smaller binary size and lower memory usage, making it suitable for embedding in various environments such as IoT devices and game engines.

<https://mruby.org/>

Recently, other implementations compatible with mruby's bytecode have been developed and are gaining attention, such as the even smaller [mruby/c](https://github.com/mrubyc/mrubyc) and [PicoRuby](https://picoruby.org/), which offers a more practical embedded ecosystem.

## Features of mruby/edge

mruby/edge is a re-implementation of the VM that executes mruby bytecode (`.mrb` files) written in Rust. It has the following features:

- **First-class WebAssembly support**: Designed to generate portable Wasm code. Currently supports compilation to `wasm32-unknown-unknown` and `wasm32-wasip1` targets, running on browsers, Wasmtime, Cloudflare Workers, Fastly Compute@Edge, and various other WASM runtimes.
- **Rust implementation**: Since the VM is written in Rust, it offers high memory safety and smooth compilation to Wasm.
- **mruby 3.2.0 compatibility**: Supports opcodes from mruby 3.2.0 through 3.4.0, enabling use of basic Ruby syntax and data types.
- **SharedMemory**: Provides a mechanism for efficient data exchange with the host environment (JavaScript, etc.) by utilizing Wasm's linear memory.

## From mruby Code to WASM

The flow of converting Ruby code to Wasm with mruby/edge is as follows:

```
Ruby source code (.rb)
    ↓ Compile with mruby compiler (mrbc, mruby-compiler2, etc.)
mruby bytecode (.mrb)
    ↓ Embed in Rust binary
    ↓ Compile to Wasm with cargo
Wasm module (.wasm)
```

This Wasm module can run in browsers. Additionally, by executing it on edge platforms such as Cloudflare Workers or Fastly, applications written in Ruby can be run at the edge.

It is also possible to embed mruby-compiler2 into Rust to directly execute Ruby scripts with mruby/edge. You can try it out on the [Playground](https://mrubyedge.github.io/playground/).

<https://mrubyedge.github.io/playground/>

## Supported Ruby Features

The following Ruby language features are supported in mruby/edge:

- Basic operations (arithmetic, comparison)
- Conditional branching (`if/elsif/else`, `case/when`)
- Method definition and recursive calls
- Class definition and instance variables
- `attr_reader` declarations
- Global variables (`$var`)
- Arrays (`Array`), Hashes (`Hash`), Strings (`String`)
- Range objects (`Range`)
- Blocks and iterators
- Exception handling (`raise`/`rescue`)
- String `unpack`/`pack` (binary data processing)

The specific supported methods can be checked in the [`COVERAGE.md`](https://github.com/mrubyedge/mrubyedge/blob/v1.1.10/mrubyedge/COVERAGE.md) file. It may also be useful to have AI reference this file.

Additionally, the following libraries are available as mrubyedge gems:

- `mruby-random`
- `mruby-regexp`
- `mruby-math`
- `mruby-serde-json`
- `mruby-time`

## Summary of mruby/edge

mruby/edge plays a crucial role as the foundation of the Uzumibi framework, enabling Ruby-written web application logic to run in edge environments.
