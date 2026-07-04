# NFT Ownership Verifier — Bot specification

**Archetype:** custom

**Voice:** professional and concise — write every user-facing message, button label, error, and empty state in this voice.

Telegram bot that verifies Ethereum NFT ownership by checking if a given wallet address owns a specific NFT (contract + token ID). Accepts addresses, ENS names, NFT links, or contract/token pairs, validates inputs, and returns verification results with on-chain proof including owner address, token metadata, and timestamp.

> This is the complete contract for the bot. Implement EVERY entry point, flow, feature, integration, and edge case below. The completeness review checks the bot against this document after each build pass.

## Primary audience

- NFT buyers/sellers
- collectors
- community moderators
- Telegram users

## Success criteria

- Accurate on-chain verification results with clear ownership status
- Rate limit enforcement with user-friendly cooldown messages
- Persistent verification history accessible via /history

## Entry points

Every feature must be reachable from the bot's command/button surface (button-first; only /start and /help are slash commands).

- **/start** (command, actor: user, command: /start) — Show usage instructions and examples
- **/help** (command, actor: user, command: /help) — Repeat usage instructions
- **/verify** (command, actor: user, command: /verify) — Upload CSV for batch verification (max 200 rows)
- **Check another** (button, actor: user, callback: verification:restart) — Initiate new verification request
- **Save as favorite** (button, actor: user, callback: favorite:add) — Placeholder for future feature
- **Report issue** (button, actor: user, callback: issue:report) — Submit error report

## Flows

### Single verification
_Trigger:_ message with NFT input

1. Parse input (address/ENS, contract, token ID, or NFT link)
2. Prompt for missing fields (one at a time)
3. Validate formats
4. Check on-chain ownership
5. Display verification result with metadata

_Data touched:_ Verification request, Verification result

### Batch verification
_Trigger:_ /verify command

1. Upload CSV file
2. Validate format and row count
3. Process batch in background
4. Send summary and downloadable results

_Data touched:_ Verification request

### Rate limit handling
_Trigger:_ User exceeds rate limit

1. Detect limit exceeded
2. Show cooldown timer and remaining time

_Data touched:_ Verification request

### History retrieval
_Trigger:_ /history command

1. Retrieve last 10 verification requests
2. Display results with timestamps

_Data touched:_ Verification request

## Data entities

Durable data (must survive a restart) uses the toolkit's persistent store, never in-memory maps.

- **User** _(retention: session)_ — Telegram user with optional email for support
  - fields: telegram_id, display_name, email
- **Verification request** _(retention: persistent)_ — User's NFT verification request and result
  - fields: request_id, telegram_id, input_wallet, contract_address, token_id, nft_link, timestamp, status, result_payload
- **Verification result** _(retention: persistent)_ — On-chain verification outcome and metadata
  - fields: owner_address, ownership_status, token_name, token_image, chain, block_number, lookup_timestamp

## Integrations

- **Telegram Bot API** (required) — User interface for commands, replies, and inline buttons
- **Blockchain data provider** (required) — Query NFT ownership and metadata (Ethereum mainnet default)
- **Admin webhook** (optional) — Notify about rate limits/abuse
Call external APIs against their real contract (correct endpoints, ids, params); credentials from env. Do not fake responses.

## Owner controls

- Configure rate limit (default 30/hour)
- Set blockchain default (Ethereum mainnet)
- Enable admin webhook for abuse alerts

## Notifications

- Rate limit reached alerts
- Admin abuse detection notifications (configurable)

## Permissions & privacy

- Telegram account authentication only
- No private key handling
- Email optional for support
- Verification data stored 30 days

## Edge cases

- Invalid address formats
- Non-ERC-721/1155 contracts
- Rate limit exhaustion
- Blockchain RPC failures
- Missing token metadata

## Required tests

- End-to-end verification flow with valid inputs
- Rate limit enforcement and cooldown display
- Error handling for invalid contract addresses
- CSV batch processing with max 200 rows

## Assumptions

- Ethereum mainnet as default blockchain
- ownerOf() is authoritative for ownership checks
- ENS names are resolved to addresses
- Metadata fetching is optional but preferred
