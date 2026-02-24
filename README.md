# Fluxora Contracts

Soroban smart contracts for the Fluxora treasury streaming protocol on Stellar. Stream USDC from a treasury to recipients over time with configurable rate, duration, and cliff.

## What's in this repo

- **Stream contract** (`contracts/stream`) — Lock USDC, accrue per second, withdraw on demand.
- **Data model** — `Stream` (sender, recipient, deposit_amount, rate_per_second, start/cliff/end time, withdrawn_amount, status).
- **Status** — Active, Paused, Completed, Cancelled.
- **Methods (stubs)** — `init`, `create_stream`, `pause_stream`, `resume_stream`, `cancel_stream`, `withdraw`, `calculate_accrued`, `get_stream_state`.

Implementation is scaffolded; storage, token transfers, and events are left for you to complete.

## Tech stack

- Rust (edition 2021)
- [soroban-sdk](https://docs.rs/soroban-sdk) (Stellar Soroban)
- Build target: `wasm32-unknown-unknown` for deployment

## Version pinning

This project pins dependencies for **reproducible builds** and **auditor compatibility**:

| Component | Version | Location | Purpose |
|---|---|---|---|
| **Rust** | 1.75 | `rust-toolchain.toml` | Ensures consistent WASM compilation |
| **soroban-sdk** | 21.7.7 | `contracts/stream/Cargo.toml` | Locked to tested Stellar Soroban network version |

When upgrading versions:

1. Update `rust-toolchain.toml` → run `rustup update` → rebuild and test
2. Update `soroban-sdk` version in Cargo.toml → update lock file → run full test suite
3. Verify compatibility with the target Stellar network (testnet, mainnet, etc.)
4. Document the change in the PR or release notes

## Local setup

### Prerequisites

- **Rust 1.75+** — Pinned in `rust-toolchain.toml` (auto-enforced via `rustup`; see [Rust version pinning](#rust-version-pinning))
- **Soroban SDK 21.7.7** — Pinned in `contracts/stream/Cargo.toml` for reproducible builds
- [Stellar CLI](https://developers.stellar.org/docs/tools/developer-tools) (optional, for deploy/test on network)

Install dependencies:

```bash
rustup update stable
rustup target add wasm32-unknown-unknown
```

Then verify:

```bash
rustc --version       # Should show 1.75 or newer
cargo --version
stellar --version     # Only if installing Stellar CLI
```

### Build

From the repo root:

```bash
cargo build --release -p fluxora_stream
```

WASM output is under `target/wasm32-unknown-unknown/release/fluxora_stream.wasm`.

### Test

```bash
cargo test -p fluxora_stream
```

This runs both:
- Unit tests in `contracts/stream/src/test.rs`
- Integration tests in `contracts/stream/tests/integration_suite.rs`

The integration suite invokes the contract with Soroban `testutils` and covers:
- `init`
- `create_stream`
- `withdraw`
- `get_stream_state`
- A full stream lifecycle from create to completed withdrawal
- Key edge cases (`init` twice, pre-cliff withdrawal, unknown stream id, underfunded deposit)

### Deploy to Stellar Testnet

> **📋 See [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md) for a complete step-by-step deployment checklist**, including build, deploy, init, and verification steps.

Quick start:

```bash
cp .env.example .env
# Edit .env with your STELLAR_SECRET_KEY, STELLAR_TOKEN_ADDRESS, STELLAR_ADMIN_ADDRESS

source .env
bash script/deploy-testnet.sh
```

Then invoke `create_stream`, `withdraw`, etc. as needed. Contract ID is saved to `.contract_id`.

## Project structure

```
fluxora-contracts/
  Cargo.toml              # workspace
  contracts/
    stream/
      Cargo.toml
      src/
        lib.rs            # contract types and impl
        test.rs           # unit tests
      tests/
        integration_suite.rs  # integration tests (Soroban testutils)
```

## Accrual formula (reference)

- **Accrued** = `min((current_time - start_time) * rate_per_second, deposit_amount)`
- **Withdrawable** = `Accrued - withdrawn_amount`
- Before `cliff_time`: withdrawable = 0.

## Related repos

- **fluxora-backend** — API and Horizon sync
- **fluxora-frontend** — Dashboard and recipient UI

Each is a separate Git repository.
