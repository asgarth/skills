# Reliability Patterns (Failover + Wait)

## 1) Configure Node Failover Up Front

`hive-tx` read helpers (`callRPC`, `callREST`) rotate across nodes based on `config.retry`.

```ts
import { config } from 'hive-tx'

config.nodes = [
  'https://api.hive.blog',
  'https://api.deathwing.me',
  'https://api.openhive.network',
  'https://rpc.mahdiyari.info'
]
config.restNodes = [
  'https://api.hive.blog',
  'https://rpc.mahdiyari.info'
]
config.timeout = 10_000
config.retry = 8
```

## 2) Broadcast With an Explicit Wait Option

Expose a public option and thread it to `tx.broadcast(waitForInclusion)`.

```ts
const result = await tx.broadcast(waitForInclusion)
```

- `waitForInclusion = false`: fast response, status usually `unknown`.
- `waitForInclusion = true`: waits for a terminal status.

## 3) Write Path Policy: Verify Before Rebroadcast

Broadcasts are not safely idempotent. Do not perform blind app-level retries.

Safer policy:

1. Use deterministic operation identifiers where possible (permlink, request IDs).
2. Prefer `waitForInclusion: true` on critical writes.
3. If broadcast errors after submit, check chain state before retrying.

## 4) Add Quorum for Critical Reads

```ts
import { callWithQuorum } from 'hive-tx'

const [account] = await callWithQuorum('condenser_api.get_accounts', [['alice']], 2)
```

Use this for authority checks, balance validation, and pre-broadcast guards.

## 5) Post-Broadcast Read Timing

Hive produces blocks about every 3 seconds. If you did not wait for inclusion, delay reads:

```ts
await new Promise((resolve) => setTimeout(resolve, 3000))
```

## 6) Minimal Status Handler

```ts
function assertTxStatus(status: string) {
  if (status === 'within_irreversible_block') return
  if (status === 'unknown') return
  throw new Error(`Transaction status not successful: ${status}`)
}
```

## 7) Common Error Mapping

- `Missing Posting Authority`: signed with wrong key, use posting.
- `Missing Active Authority`: signed with wrong key, use active.
- `Please wait to transact, or power up`: low RC.
- `Duplicate transaction check failed`: tx may already be known; verify chain state first.
