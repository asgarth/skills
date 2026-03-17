# Hive TX API Reference

## Install and Import

```bash
npm install hive-tx
```

```ts
import { config, callRPC, callREST, callWithQuorum, Transaction, PrivateKey } from 'hive-tx'
```

## Node Config With Automatic Failover

`hive-tx` failover is built into `callRPC`/`callREST`. Configure node arrays once:

```ts
import { config } from 'hive-tx'

export function configureHiveNodes() {
  config.nodes = [
    'https://api.hive.blog',
    'https://api.deathwing.me',
    'https://api.openhive.network',
    'https://rpc.mahdiyari.info',
    'https://techcoderx.com',
    'https://hiveapi.actifit.io',
    'https://api.c0ff33a.uk'
  ]

  config.restNodes = [
    'https://api.hive.blog',
    'https://rpc.mahdiyari.info',
    'https://techcoderx.com',
    'https://hiveapi.actifit.io',
    'https://api.c0ff33a.uk'
  ]

  config.timeout = 10_000
  config.retry = 8
}
```

## Read APIs

There is no `Client` object in `hive-tx`; use module functions directly.

```ts
import { callRPC } from 'hive-tx'

const [account] = await callRPC('condenser_api.get_accounts', [['alice']])
const post = await callRPC('condenser_api.get_content', ['author', 'permlink'])
const props = await callRPC('condenser_api.get_dynamic_global_properties', [])

const history = await callRPC('condenser_api.get_account_history', [
  'alice',
  -1,
  100
])

const rc = await callRPC('rc_api.find_rc_accounts', { accounts: ['alice'] })
```

## Quorum Reads for Critical Checks

Use quorum on important pre-broadcast checks:

```ts
import { callWithQuorum } from 'hive-tx'

const [account] = await callWithQuorum('condenser_api.get_accounts', [['alice']], 2)
```

## REST APIs

Use `callREST` only against nodes that support those endpoints:

```ts
import { callREST } from 'hive-tx'

const balances = await callREST('balance', '/accounts/{account-name}/balances', {
  'account-name': 'alice'
})
```

## Transaction Lifecycle

```ts
import { Transaction, PrivateKey } from 'hive-tx'

const tx = new Transaction()
await tx.addOperation('vote', {
  voter: 'alice',
  author: 'bob',
  permlink: 'my-post',
  weight: 10_000
})

tx.sign(PrivateKey.from(process.env.HIVE_POSTING_KEY as string))

const fast = await tx.broadcast(false) // { tx_id, status: "unknown" }
const waited = await tx.broadcast(true) // waits for final status
```

## Status and Verification

- `tx.broadcast(false)`: returns quickly; status is usually `unknown`.
- `tx.broadcast(true)`: waits until a terminal status is reached.
- `tx.checkStatus()`: poll status using the transaction id calculated from the signed tx.
