# Universal Archetype Adaptation Matrix (API Design)

The 8 invariants adapt dynamically across every software archetype by abstracting tools to universal discrete roles:

| Archetype | Operations & Verbs | Schema Definition | Canonical Errors | Idempotency Mechanism | Pagination & Bounds |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **REST / HTTP Web APIs** | Resource paths + HTTP methods (`POST /orders`) | OpenAPI 3.1, JSON Schema | RFC 9457 Problem Details (`type`, `title`, `status`) | `Idempotency-Key` header with in-flight lock | Keyset / cursor tokens (`page_token`), `limit` capped at 100 |
| **gRPC / Protocol Buffers** | RPC methods (`rpc CreateOrder()`) | `.proto` service & message defs | `google.rpc.Status` with rich `ErrorInfo` details | Request UUID in message payload or metadata | Keyset pagination in message request/response |
| **GraphQL / Data Graphs** | Queries, Mutations, Subscriptions | GraphQL SDL (`type`, `input`) | Top-level `errors[]` with `extensions.code` | Client mutation ID / unique token | Relay Connection spec (`edges`, `node`, `cursor`, `pageInfo`) |
| **Event-Driven / Messaging** | Topics, Queues, Event types (`order.created.v1`) | AsyncAPI, CloudEvents, Avro, Protobuf | Dead Letter Queues (DLQ), poison message envelopes | Message deduplication ID / Event idempotency keys | Batch sizes, streaming backpressure, partition keys |
| **In-Process SDKs / Libraries** | Functions, Classes, Methods, Interfaces | TypeScript types, Rust traits, Python Protocols | Typed Result types (`Result<T, E>`), domain exceptions | Pure functions or deterministic state machines | Iterators, Generators, Streams, bounded slices |
| **CLI / TUI Command Lines** | Commands, Subcommands, Flags, Positional args | CLI schemas (POSIX, GNU, clap, argparse) | Standard exit codes (`0` vs `1..255`), structured `stderr` | Re-runnable commands (`--force`, `--dry-run`, state checks) | Streaming stdout, `--limit`, interactive paging |
| **Embedded & Systems IPC** | Unix sockets, ioctl, shared memory, D-Bus | C ABIs, FlatBuffers, Cap'n Proto | Linux errno codes, packed status integers | Hardware transaction sequence numbers | Bounded ring buffers, chunked memory windows |
| **AI Agent Tools / MCP** | Tool names, action verbs, input schemas | JSON Schema tool declarations | Tool error responses (`isError: true`, structured error text) | Tool execution tokens / persistent run-states | Bounded tool call outputs, truncated token limits |
