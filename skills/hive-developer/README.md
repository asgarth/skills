# hive-developer

Hive blockchain development skill focused on the `hive-tx` npm library.

## Origin

This skill was bootstrapped from the structure and coverage of
[hive-master-skill](https://github.com/Mantequilla-Soft/hive-master-skill).
Unlike the original skill, which was focused on Claude usage, this skill is designed to work across coding agents: Codex, OpenCode, and Claude.

## Why `hive-tx`

We chose `hive-tx` as a more modern and versatile implementation target for Hive app development:

- First-class RPC + REST access in one SDK, including `callREST(...)`
- Automatic node fallback through multi-node config (`config.nodes`, retries)
- Quorum-based reads through `callWithQuorum(...)` for critical checks
- Unified transaction flow (`Transaction` + optional wait via `broadcast(true)`)

## What This Skill Covers

- JavaScript/TypeScript Hive development with `hive-tx`
- Automatic failover via `config.nodes` + retries
- REST API access through `callREST(...)`
- Quorum reads through `callWithQuorum(...)`
- Safe transaction broadcasting with status-aware handling
- Optional wait-for-inclusion via `tx.broadcast(true)`
- Posting/active key selection and multi-key signing patterns
- Polling and real-time block/operation processing patterns

## Main Entry

- `SKILL.md`

## Reference Files

- `api-reference.md`
- `operations.md`
- `reliability.md`
- `keys-and-auth.md`
- `streaming-and-realtime.md`
