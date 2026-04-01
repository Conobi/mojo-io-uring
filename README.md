# mojo-io-uring

A Linux [`io_uring`](https://unixism.net/loti/) userspace interface for
[Mojo](https://docs.modular.com/mojo/).

> [!WARNING]
> This project is under active development and is **not stable**.
> APIs may change without notice between commits.

Only x86_64 Linux is supported.

## Packages

| Package | Description |
|---|---|
| `linux_raw` | Raw Linux syscall bindings — C type aliases, errno codes, `io_uring` structs, and direct syscall wrappers (x86_64) |
| `mojix` | Safe I/O wrappers — file descriptor traits, network helpers, error handling via `Errno` |
| `io_uring` | High-level async I/O ring — memory-mapped SQ/CQ, fluent operation builders, ring-mapped buffers |
| `event_loop` | Callback-driven event loop built on `io_uring` with a `CompletionHandler` trait |

## Getting Started

Requires [`uv`](https://docs.astral.sh/uv/) on Linux x86_64.

```bash
uv sync                                # install dev dependencies (mojox provides the Mojo compiler)
uv run -- bash scripts/build.sh        # build all .mojopkg files
uv run -- bash scripts/run_tests.sh    # run all tests
uv run -- mojo run -I . examples/reaction_timer.mojo  # run an example
```

> [!TIP]
> To run a single test:
> ```bash
> uv run -- mojo run -I . -D ASSERT=all tests/io_uring/test_nop.mojo
> ```

## Architecture

Three-layer abstraction with an event loop on top:

```
event_loop/    ← callback-driven event loop
io_uring/      ← high-level async I/O ring
mojix/         ← safe I/O wrappers
linux_raw/     ← raw Linux syscall bindings (x86_64)
```

<details>
<summary>Key design choices</summary>

- **Compile-time generics** — Entry sizes (`SQE64`/`SQE128`, `CQE16`/`CQE32`) and polling mode are type parameters on `IoUring`
- **RAII file descriptors** — `OwnedFd[is_registered]` auto-closes on destruction
- **Operation builder pattern** — Op types (`Nop`, `Accept`, `Read`, `Write`, `Send`, `SendMsg`, `RecvMsg`, …) use method chaining
- **Origin system** — Mojo's borrow checker (`MutableOrigin`, `ImmutableOrigin`) for memory safety without GC

</details>

## License

Licensed under the Apache License v2.0 with LLVM Exceptions
(see [LICENSE](LICENSE)).
