# Keys and Signing Rules

## Key Hierarchy

- `posting`: social/content actions (`vote`, `comment`, most `custom_json`).
- `active`: wallet/governance actions (`transfer`, vesting, witness/proposal actions).
- `memo`: encrypt/decrypt transfer memos.
- `owner`: account recovery and key rotation only; avoid in app runtime.

## Create Key Objects

```ts
import { PrivateKey } from 'hive-tx'

const postingKey = PrivateKey.from(process.env.HIVE_POSTING_KEY as string)
const activeKey = PrivateKey.from(process.env.HIVE_ACTIVE_KEY as string)
```

## Sign With Required Authority

```ts
import { Transaction } from 'hive-tx'

const tx = new Transaction()
await tx.addOperation('vote', {
  voter: 'alice',
  author: 'bob',
  permlink: 'post-permlink',
  weight: 10_000
})

tx.sign(postingKey)
```

Mixed authorities in one tx:

```ts
tx.sign([postingKey, activeKey])
```

## Authority Mapping Helper

```ts
const postingOps = new Set([
  'vote',
  'comment',
  'delete_comment',
  'claim_reward_balance',
  'custom_json'
])

const activeOps = new Set([
  'transfer',
  'transfer_to_vesting',
  'withdraw_vesting',
  'delegate_vesting_shares',
  'recurrent_transfer',
  'account_witness_vote',
  'update_proposal_votes'
])
```

Use this before signing to fail early on key mismatch.

## Verify Key Against On-Chain Authorities

```ts
import { callRPC, PrivateKey } from 'hive-tx'

async function isPostingKeyValid(accountName: string, postingWif: string) {
  const [account] = await callRPC('condenser_api.get_accounts', [[accountName]])
  const pub = PrivateKey.from(postingWif).createPublic().toString()
  return account.posting.key_auths.some(([k]: [string, number]) => k === pub)
}
```

## Security Defaults

- Never hardcode private keys.
- Use env vars for local dev and secret managers in production.
- Redact keys and memo plaintext from logs.
- Keep owner keys offline.
