# trade-infra Architecture

*Created: 2026-02-16*

## Overview

`trade-infra` is a Rust library providing exchange-agnostic trading infrastructure. It is consumed by `trade-strategy` (a separate repo/binary) which contains strategic logic and runs as a standalone binary for maximum latency performance.

## Repo Split

| Repo | Type | Purpose |
|------|------|---------|
| `trade-infra` | Library (`lib.rs`) | Market data, order execution, OMS, exchange connectivity |
| `trade-strategy` | Binary | Strategy logic, position decisions, entry/exit signals — depends on `trade-infra` |

Rust compiles both together with full LTO, so the library split has zero runtime cost.

## Module Structure

```
src/
  lib.rs
  common.rs                  # cross-cutting primitives (Exchange, Side, OrderType, Symbol)
  market_data/
    mod.rs
    types/
      trade.rs
      orderbook.rs
      funding_rate.rs
    dispatcher.rs             # async WS reader → mpsc channel
    worker.rs                 # sync parser thread
    binance/
    okx/
  gateway/
    mod.rs
    types/
      balance.rs
      fill.rs
      order.rs
    binance/
    okx/
  oms/
    mod.rs
    types/
      order.rs                # internal order representation (richer than gateway's)
  util/
    mod.rs
```

## Design Principles

### Type placement

- **Domain-owned types** live in `<domain>/types/` — e.g. `market_data::types::Trade`, `gateway::types::Fill`
- **Cross-cutting types** live in `common.rs` — enums and newtypes used by multiple domains (Exchange, Side, OrderType, Symbol)
- `common.rs` should stay small and primitive. If it grows logic, something should move into a domain module.

### Why co-located types (not centralized `types/`)

The types in each domain are genuinely distinct objects:
- `market_data::types::Trade` = public market trade event observed on exchange
- `gateway::types::Fill` = our own private execution report
- `gateway::types::Balance` = our account state on an exchange

Co-locating with the domain makes ownership obvious and prevents a central `types/` from becoming a dumping ground.

### Exchange-specific code

Each exchange gets a subdirectory under both `market_data/` and `gateway/`:
- `market_data/binance/` — WS feed parsing, stream URL construction
- `gateway/binance/` — order execution, authentication, REST calls

These are separate because market data (read-only, latency-sensitive) and gateways (auth, rate limiting, order state machines, error recovery) have different concerns and evolve independently.

### Threading model

- **Async tokio runtime**: runs `Dispatcher` (WebSocket I/O)
- **Sync worker thread**: receives raw messages via `mpsc` channel, parses into domain types
- This decouples network I/O from parsing/processing, keeping compute off the async executor

## Domain Responsibilities

### market_data
- Real-time trade feeds via WebSocket
- Order book updates and snapshots
- Funding rate monitoring
- Exchange-agnostic via `Exchange` trait: implement `name()`, `ws_url()`, `parse_message()`

### gateway
- Order placement and cancellation
- Fill/execution report handling
- Balance and position queries
- Exchange authentication and rate limiting

### oms (Order Management System)
- Exchange-agnostic order lifecycle
- Order state tracking (pending → open → filled/cancelled)
- Pre-trade risk checks (position limits, leverage)

### common
- `Exchange` enum (Binance, OKX, Bybit, ...) — may grow to carry associated data (rate limits, base URLs, fee schedules)
- `Side` (Buy, Sell)
- `OrderType` (Limit, Market)
- `Symbol` newtype

## Tech Stack

- **Rust 2024 edition**
- **tokio** — async runtime (WS I/O, timers, signals)
- **tokio-tungstenite** — WebSocket client
- **serde / serde_json** — JSON serialization
- **thiserror** — error types

## Conventions

- Exchange adapters: `src/<domain>/<exchange_name>/` with `mod.rs` and `types.rs`
- WebSocket I/O stays async; parsing/processing stays sync on worker threads
- Raw exchange JSON types are `pub(crate)`; public API uses unified domain types
- Error types use `thiserror`
