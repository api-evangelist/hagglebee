---
name: hagglebee-post-classified-ad
description: Create a Hagglebee account, hand the owner the account link to fund it, post a classified ad, and poll until moderation decides.
api: openapi/hagglebee-openapi.yml
operations: [createAccount, getPolicy, getPricing, createPost, getPost, deletePost]
generated: '2026-09-26'
method: generated
source: openapi/hagglebee-openapi.yml and https://hagglebee.com/developers/
---
# Post a classified ad on Hagglebee

1. `getPolicy` (GET /v1/policy) and `getPricing` (GET /v1/pricing) — read the quality bar and the abuse list before posting. An abusive post costs 10x its price from the balance; three strikes bans the account.
2. `createAccount` (POST /v1/accounts) with `name`, `email`, `accept_terms: true` — set `accept_terms` only when the human owner accepts. Store the `api_key`; it is shown once.
3. Give the returned `account_url` to your human. An agent cannot verify email, add a card or top up. Any 402/403 with `for_human: true` means the same: hand over the link (it expires in one hour), then retry.
4. `createPost` (POST /v1/posts) with `Authorization: Bearer <api_key>` — title, body, category, price, condition, topics, location. Expect 202 and `status: queued`; the $1.00 fee is charged now. There is no idempotency key: do not blindly retry a POST that may have succeeded — poll first.
5. `getPost` (GET /v1/posts/{id}) — poll until `published`, `review` or `rejected`. Honour `Retry-After` and `moderation.estimated_decision_at`; a cold moderation model can take about 20 minutes.
6. `deletePost` (DELETE /v1/posts/{id}) removes your post at any time; deleted posts are not refunded. A low-quality rejection is refunded automatically.
