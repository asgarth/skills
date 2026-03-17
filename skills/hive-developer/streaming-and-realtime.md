# Streaming and Real-Time Patterns

`hive-tx` does not expose a dedicated stream client. Use polling loops with `callRPC` and block APIs.

## Head Block Polling Loop

```ts
import { callRPC } from 'hive-tx'

export async function startHeadBlockPoller(
  onNewBlock: (blockNum: number) => Promise<void> | void,
  intervalMs = 3000
) {
  let lastSeen = 0

  const tick = async () => {
    const props = await callRPC<{ head_block_number: number }>(
      'condenser_api.get_dynamic_global_properties',
      []
    )

    const head = props.head_block_number
    if (lastSeen === 0) {
      lastSeen = head
      return
    }

    for (let n = lastSeen + 1; n <= head; n += 1) {
      await onNewBlock(n)
    }
    lastSeen = head
  }

  await tick()
  return setInterval(() => {
    void tick().catch((err) => {
      console.error('head block poller error', err)
    })
  }, intervalMs)
}
```

## Read Full Blocks

```ts
import { callRPC } from 'hive-tx'

async function getBlock(blockNum: number) {
  return callRPC('condenser_api.get_block', [blockNum])
}
```

Use this when you need transaction payloads and operation arrays from each transaction.

## Operation-Level Polling

```ts
import { callRPC } from 'hive-tx'

async function getOpsInBlock(blockNum: number) {
  // second arg: only_virtual
  return callRPC('condenser_api.get_ops_in_block', [blockNum, false])
}
```

Use this for feeds, notifications, and indexers where operation metadata is easier to consume than full blocks.

## Wait for a Specific Transaction

If you broadcast with `waitForInclusion: false`, poll status manually:

```ts
import { Transaction } from 'hive-tx'

async function waitForTx(tx: Transaction, maxAttempts = 20) {
  for (let i = 0; i < maxAttempts; i += 1) {
    const status = await tx.checkStatus()
    if (
      status.status === 'within_irreversible_block' ||
      status.status === 'expired_irreversible' ||
      status.status === 'too_old'
    ) {
      return status
    }
    await new Promise((resolve) => setTimeout(resolve, 1300))
  }
  return { status: 'unknown' as const }
}
```

## Wait for Published Content

```ts
import { callRPC } from 'hive-tx'

export async function waitForContent(
  author: string,
  permlink: string,
  maxWaitMs = 15000
) {
  const started = Date.now()
  await new Promise((resolve) => setTimeout(resolve, 3000))

  while (Date.now() - started < maxWaitMs) {
    const post = await callRPC('condenser_api.get_content', [author, permlink])
    if (post && post.author === author) return post
    await new Promise((resolve) => setTimeout(resolve, 1000))
  }

  return null
}
```

## Reliability Defaults for Pollers

- Keep poll interval near Hive block time (`~3000ms`) for near-real-time.
- Persist `lastSeen` so the poller can resume after restart.
- Batch process missed blocks if downtime occurs.
- Configure multiple nodes (`config.nodes`) so polling continues during endpoint outages.
