# Hive Operations with hive-tx

Build transactions with `addOperation`, then sign and broadcast.

## Reusable Broadcast Helper (Optional Wait)

```ts
import { Transaction } from 'hive-tx'

type BroadcastOptions = {
  waitForInclusion?: boolean
  requireIrreversible?: boolean
}

export async function broadcastTx(tx: Transaction, options: BroadcastOptions = {}) {
  const waitForInclusion = options.waitForInclusion ?? false
  const requireIrreversible = options.requireIrreversible ?? false
  const result = await tx.broadcast(waitForInclusion)

  if (requireIrreversible && result.status !== 'within_irreversible_block') {
    throw new Error(
      `Transaction ${result.tx_id} finished with status ${result.status}`
    )
  }

  return result
}
```

## Vote (Posting Key)

```ts
import { Transaction, PrivateKey } from 'hive-tx'

const tx = new Transaction()
await tx.addOperation('vote', {
  voter: 'alice',
  author: 'bob',
  permlink: 'post-permlink',
  weight: 10_000
})

tx.sign(PrivateKey.from(process.env.HIVE_POSTING_KEY as string))
const result = await broadcastTx(tx, { waitForInclusion: true })
```

## Post (Comment + Comment Options in Same Tx)

```ts
import { Transaction, PrivateKey } from 'hive-tx'

const beneficiaries = [
  { account: 'alice', weight: 500 },
  { account: 'threespeakfund', weight: 1000 }
].sort((a, b) => a.account.localeCompare(b.account))

const tx = new Transaction()

await tx.addOperation('comment', {
  parent_author: '',
  parent_permlink: 'hive-123456',
  author: 'myaccount',
  permlink: 'my-post-permlink',
  title: 'My Post',
  body: 'Post body in markdown',
  json_metadata: JSON.stringify({
    app: 'myapp/1.0.0',
    tags: ['hive', 'dev']
  })
})

await tx.addOperation('comment_options', {
  author: 'myaccount',
  permlink: 'my-post-permlink',
  max_accepted_payout: '1000000.000 HBD',
  percent_hbd: 10_000,
  allow_votes: true,
  allow_curation_rewards: true,
  extensions: [[0, { beneficiaries }]]
})

tx.sign(PrivateKey.from(process.env.HIVE_POSTING_KEY as string))
const result = await broadcastTx(tx, { waitForInclusion: true })
```

## Reply (Posting Key)

```ts
const tx = new Transaction()
await tx.addOperation('comment', {
  parent_author: 'parent-author',
  parent_permlink: 'parent-permlink',
  author: 'myaccount',
  permlink: 'my-reply-permlink',
  title: '',
  body: 'Reply body',
  json_metadata: JSON.stringify({ app: 'myapp/1.0.0' })
})
```

## Transfer and Wallet Operations (Active Key)

```ts
import { Transaction, PrivateKey } from 'hive-tx'

const tx = new Transaction()

await tx.addOperation('transfer', {
  from: 'sender',
  to: 'receiver',
  amount: '1.000 HIVE',
  memo: 'Thanks'
})

await tx.addOperation('transfer_to_vesting', {
  from: 'sender',
  to: 'sender',
  amount: '5.000 HIVE'
})

tx.sign(PrivateKey.from(process.env.HIVE_ACTIVE_KEY as string))
const result = await broadcastTx(tx, { waitForInclusion: true })
```

## Custom JSON (Posting Key)

```ts
import { Transaction, PrivateKey } from 'hive-tx'

const tx = new Transaction()
await tx.addOperation('custom_json', {
  required_auths: [],
  required_posting_auths: ['alice'],
  id: 'follow',
  json: JSON.stringify([
    'follow',
    { follower: 'alice', following: 'bob', what: ['blog'] }
  ])
})

tx.sign(PrivateKey.from(process.env.HIVE_POSTING_KEY as string))
const result = await broadcastTx(tx, { waitForInclusion: true })
```

## Community Subscribe (Custom JSON, Posting Key)

```ts
const tx = new Transaction()
await tx.addOperation('custom_json', {
  required_auths: [],
  required_posting_auths: ['alice'],
  id: 'community',
  json: JSON.stringify(['subscribe', { community: 'hive-123456' }])
})
```

## Mixed Authorities (Posting + Active)

If one transaction includes posting and active operations, sign with both keys:

```ts
import { Transaction, PrivateKey } from 'hive-tx'

const tx = new Transaction()
await tx.addOperation('vote', {
  voter: 'alice',
  author: 'bob',
  permlink: 'post-permlink',
  weight: 5000
})
await tx.addOperation('transfer', {
  from: 'alice',
  to: 'bob',
  amount: '0.100 HIVE',
  memo: 'tip'
})

const posting = PrivateKey.from(process.env.HIVE_POSTING_KEY as string)
const active = PrivateKey.from(process.env.HIVE_ACTIVE_KEY as string)
tx.sign([posting, active])

const result = await broadcastTx(tx, { waitForInclusion: true })
```

## Status Values to Handle

- `unknown`: broadcast submitted without waiting.
- `within_irreversible_block`: transaction included and irreversible.
- `expired_irreversible`: transaction expired before irreversible inclusion.
- `too_old`: status tracking window passed.
