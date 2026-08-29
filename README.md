# VLAD CHAIN

> **The RWA Layer 3 for the Robinhood Chain**

VLAD CHAIN is an experimental full-stack blockchain and Real World Asset (RWA)
network simulation. It combines a terminal-style block explorer, simulated token
transfers, a synthetic RWA registry, AI validator personas, and an
AI-assisted governance system in one interactive web application.

The project is designed to demonstrate what an AI-governed RWA settlement
network could look and feel like. It is **not a live blockchain**, does not
custody real assets, and does not settle real financial transactions.

**Contract address:** Coming soon

---

## Table of Contents

- [What the project includes](#what-the-project-includes)
- [What is simulated](#what-is-simulated)
- [Application walkthrough](#application-walkthrough)
- [System architecture](#system-architecture)
- [Blockchain simulation](#blockchain-simulation)
- [Transaction lifecycle](#transaction-lifecycle)
- [Wallets and faucet](#wallets-and-faucet)
- [RWA registry](#rwa-registry)
- [AI validator council](#ai-validator-council)
- [Governance and GIPs](#governance-and-gips)
- [Data storage](#data-storage)
- [Development and serverless backends](#development-and-serverless-backends)
- [Technology stack](#technology-stack)
- [Project structure](#project-structure)
- [Getting started](#getting-started)
- [Configuration](#configuration)
- [Available scripts](#available-scripts)
- [REST API reference](#rest-api-reference)
- [Testing and validation](#testing-and-validation)
- [Deployment](#deployment)
- [Security and limitations](#security-and-limitations)
- [Contributing](#contributing)
- [License](#license)

---

## What the project includes

VLAD CHAIN currently ships as a working web application with:

- A responsive React terminal interface for desktop and mobile
- A continuously running in-memory blockchain simulation
- Solana-inspired slots and epochs with approximately 400 ms block intervals
- Account balances, a pending transaction pool, blocks, and transaction history
- Transaction receipts with a TX ID, block height, UTC timestamp, amount, and fee
- TX ID search in the explorer
- A test-token faucet with cooldown and daily limits
- Simulated wallet generation and support for browser wallet connection flows
- A synthetic registry of tokenized equities, treasuries, real estate,
  commodities, and private credit
- Six AI validator personas with individual and multi-agent chat experiences
- VladChain Improvement Proposals (GIPs), debate transcripts, voting data, and
  archived governance history
- SQLite persistence for wallets, transactions, chat messages, GIPs, and the
  network clock
- A serverless API variant for Vercel deployments
- A Solidity ERC-20 contract source for the future VLAD CHAIN token

## What is simulated

The application intentionally uses simulated data and behavior:

- RWA prices, yields, TVL, custodians, issuers, compliance labels, oracle names,
  and proof-of-reserve attestations are demonstration data.
- The AI validator council is a collection of LLM personas. It does not provide
  Byzantine fault tolerance or cryptographic consensus.
- Blocks, hashes, addresses, wallet phrases, balances, and transactions are
  generated for the application and do not exist on a public blockchain.
- The RWA registry does not represent ownership of real securities, treasuries,
  property, commodities, or credit instruments.
- The Solidity contract is source code only. It is not connected to the
  simulation and no deployed contract address is currently advertised.
- Compliance badges are descriptive UI fields, not real KYC/AML enforcement or
  legal approval.

This distinction is important: VLAD CHAIN is a product prototype and research
environment, not an investment product or production settlement network.

---

## Application walkthrough

The frontend uses a single-page, tab-based terminal interface.

### HOME

The landing page introduces the network, displays the current epoch and slot,
links to the project’s X and GitHub pages, and provides terminal-style shortcuts
to the major features. It also includes the RWA Layer 3 story, system activity,
and live AI debate content.

### EXPLORER

The explorer exposes the current simulated network state:

- Recent blocks and full block history
- Block height, producer, transaction count, and timestamp
- Network accounts and balances
- Active validator names and statistics
- Pending and confirmed transactions
- Transaction search by exact TX ID

Desktop users receive table-oriented views. Smaller screens use responsive,
readable layouts and horizontally contained data where necessary.

### FAUCET

The faucet sends simulated VLAD CHAIN test tokens to a network address. Requests
are subject to application-level limits so repeated requests cannot mint without
restriction.

### SEND

The send screen transfers simulated tokens between known accounts. A successful
transaction displays a receipt containing:

- TX ID
- Block height
- UTC timestamp
- Sender and recipient
- Amount
- Network fee

The TX ID can subsequently be searched from the explorer.

### ORACLE

The Oracle screen lets users talk with individual AI validator personalities.
Responses depend on the configured AI provider and the persona’s system prompt.
The feature is a conversational interface—not a live price oracle.

### RWA

The RWA dashboard summarizes the synthetic asset registry with:

- Total simulated value tokenized
- Number of assets and asset classes
- TVL by asset class
- Weighted yield for yielding assets
- Price and 24-hour movement
- Compliance labels
- Proof-of-reserve status and attestation metadata
- Search and asset-class filtering

### GIPs

The GIP interface presents VladChain Improvement Proposals through several
states, including draft, debating, voting, and archived. Users can inspect
proposal summaries, full descriptions, tags, priorities, validator messages,
votes, and final outcomes.

### DOCS

The in-app documentation provides a compact terminal-native introduction to the
protocol concept and its API. This README and
[`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md) provide the deeper technical
reference.

---

## System architecture

```text
┌──────────────────────────────────────────────────────────────────┐
│                    React + TypeScript frontend                    │
│                                                                  │
│  Home · Explorer · Faucet · Send · Oracle · RWA · GIPs · Docs   │
└──────────────────────────────┬───────────────────────────────────┘
                               │ REST + polling
                               ▼
┌──────────────────────────────────────────────────────────────────┐
│                    Express + TypeScript API                      │
│                                                                  │
│  Chain simulation · Transactions · AI personas · GIPs · RWA     │
└───────────────┬─────────────────────┬────────────────────────────┘
                │                     │
                ▼                     ▼
┌────────────────────────┐   ┌─────────────────────────────────────┐
│ SQLite                  │   │ Optional external AI providers     │
│                         │   │                                     │
│ wallets · transactions │   │ Anthropic Claude · OpenAI · Groq   │
│ GIPs · chat · slot     │   │                                     │
└────────────────────────┘   └─────────────────────────────────────┘
```

The frontend polls REST endpoints rather than relying on WebSockets. During
development, Vite runs on port `5000` and proxies API requests to the Express
server on port `4000`.

### Request flow

1. The React client makes a relative request such as `POST /api/send`.
2. The Express server validates the request and updates the chain simulation.
3. Important records are written to SQLite.
4. The server returns JSON to the client.
5. React updates the relevant terminal panel without a full page reload.

---

## Blockchain simulation

The development backend models a lightweight chain in memory.

### Timing model

| Property | Current behavior |
|---|---|
| Target slot/block interval | Approximately 400 ms |
| Slots per epoch | 432,000 |
| Epoch clock | Persisted in SQLite |
| Block production | Continuous timer in the development backend |
| Simulated activity | Periodic generated transfers |

### Accounts

Each account has an address and a simulated VLAD CHAIN balance. The chain starts
with named system/validator accounts, and user-generated wallets can be restored
from SQLite when the server restarts.

### Transactions

A transfer includes:

- Unique generated hash/TX ID
- Sender
- Recipient
- Amount
- Fee
- Creation timestamp
- Block height after inclusion

The development backend requires both accounts to exist, requires a positive
amount, and checks that the sender can cover the amount plus the fee.

### Blocks and rewards

When a block is produced, the pending transaction pool is drained into that
block. Included transactions receive the new block height. The simulated block
producer receives a fixed reward plus collected transaction fees.

### Persistence boundary

The simulation is deliberately hybrid:

- Wallet records, transaction records, and the slot clock persist.
- The complete in-memory block list, account state, and pending pool are not a
  durable production ledger.

Restart behavior should therefore not be interpreted as blockchain-grade state
recovery.

---

## Transaction lifecycle

The development backend’s send flow is:

```text
POST /api/send
       │
       ▼
Validate sender, recipient, amount, balance, and fee
       │
       ▼
Update simulated balances and create a transaction
       │
       ├──────────────► Log transaction to SQLite
       │
       ▼
Place transaction in pending pool
       │
       ▼
Next block includes transaction and stamps block height
       │
       ▼
Return receipt to the frontend
       │
       ▼
Search later with GET /api/tx/:txId
```

The send endpoint waits briefly—up to two seconds—for normal block inclusion so
the receipt can usually include its block height immediately.

### Development fee

The long-running backend currently charges a flat `0.001` simulated VLAD CHAIN
fee per transfer. This is not a dynamic fee market.

---

## Wallets and faucet

### Generated wallets

`POST /api/generate_wallet` creates a simulation-only address and a 12-word
recovery-style phrase, stores the wallet in SQLite, and adds the account to the
network.

These values are generated for the demo. They are **not guaranteed to be
cryptographically secure BIP-39 wallets** and must never be reused for real
funds.

### Browser wallets

The frontend also includes connection components for common injected wallet
providers, including MetaMask-style EVM wallets and Phantom-style Solana
wallets. Connecting a browser wallet provides an address to the UI; it does not
turn the simulated ledger into a real on-chain transaction system.

### Faucet protections

The development faucet currently applies:

- A 30-second cooldown
- A maximum of two requests per day
- A daily maximum of 1,000 simulated tokens

The faucet is for development and demonstration balances only.

---

## RWA registry

The RWA registry is a curated synthetic dataset with light deterministic price
jitter to make the dashboard feel live. Values change approximately every
minute.

### Asset classes and examples

| Asset class | Example symbols | Demonstration purpose |
|---|---|---|
| Tokenized Equities | `vHOOD`, `vSPY`, `vNVDA` | Equity-like prices and issuer metadata |
| US Treasuries | `vTBILL`, `vUST10` | NAV and yield-bearing government debt examples |
| Real Estate | `vNYCRE`, `vRESI` | Property-pool and REIT-style assets |
| Commodities | `vXAU`, `vWTI` | Vaulted-gold and energy-basket examples |
| Private Credit | `vCRED`, `vTRADE` | Lending-pool and receivables examples |

### Asset fields

Each registry item includes:

- Internal ID, symbol, and display name
- Asset class
- Simulated issuer and custodian
- Reference price and annualized yield
- Simulated TVL
- Compliance labels
- Attestor and attestation cadence
- Oracle description
- Chain label and decimals
- Generated current price and 24-hour change
- Proof-of-reserve object with status, collateralization percentage, attestor,
  timestamp, and cadence

### Registry statistics

`GET /api/rwa/stats` calculates:

- Total simulated value tokenized
- Asset count
- Asset-class count
- TVL-weighted average yield
- Proof-of-reserve coverage
- TVL and count grouped by asset class
- Last update timestamp

All organizations, attestations, prices, yields, and values in this registry are
fictional or illustrative unless explicitly stated otherwise.

---

## AI validator council

The canonical validator personalities are defined in
`backend/src/multi-agent.ts`.

| Validator | Persona | Perspective |
|---|---|---|
| **Alice** | The Origin Validator | Reflective decisions grounded in the network’s history |
| **Ayra** | The Speculative Economist | Incentive design, market structure, and RWA economics |
| **Jarvis** | The Existentialist | Ownership, identity, recursion, and philosophical trade-offs |
| **Cortana** | The Protocol Engineer | Technical implementation and protocol clarity |
| **Lumina** | The Ethical One | Fairness, access, and human impact |
| **Nix** | The Chaotic One | Adversarial scenarios, edge cases, and stress testing |

### How AI calls work

- The multi-agent personality implementation currently calls Claude when the
  required Claude key is configured.
- Separate OpenAI, Claude, and Groq-compatible provider modules exist for
  different application features.
- Narrative generation uses the Groq-compatible provider path and can return a
  deterministic local fallback when the provider is unavailable.
- Provider errors do not stop the blockchain simulation itself.

The project does not currently implement automatic multi-provider failover or
route every validator to a different model.

---

## Governance and GIPs

GIPs are structured governance records stored in SQLite.

### Proposal data

A proposal can contain:

- ID, title, author, category, and priority
- Summary and full proposal
- Current status
- Creation and update timestamps
- Tags
- Debate messages
- Validator votes
- Final decision

### Lifecycle

```text
Draft → Debating → Voting → Approved/Rejected → Archived
```

The server seeds realistic demonstration proposals and gradually releases debate
content to create a live governance experience. This is an application
simulation, not binding on-chain governance.

---

## Data storage

The development backend uses `better-sqlite3`. Its database is stored at:

```text
backend/data/vladchain.db
```

### Tables

| Table | Purpose |
|---|---|
| `chat_messages` | Terminal chat and system activity |
| `gips` | Governance proposal records |
| `gip_messages` | Validator debate messages |
| `slot_data` | Persisted slot and epoch clock |
| `wallets` | Generated simulation wallets |
| `transactions` | TX ID, block height, addresses, amount, fee, and timestamps |

Although configuration code recognizes PostgreSQL-style settings, the current
implementation falls back to SQLite. PostgreSQL should not be considered a
finished production storage option yet.

---

## Development and serverless backends

The repository has two API implementations:

### `backend/src/`

The primary local-development backend:

- Long-running Express process
- Continuous slot and block timers
- 0.001 transfer fee, checked and deducted from sender balance
- Send endpoint waits for inclusion before returning its receipt
- Multi-agent router is mounted
- SQLite persistence is available throughout the process

### `api/`

The Vercel-oriented serverless variant:

- Request-driven execution instead of continuous timers
- Synchronous block stamping during transaction requests
- Different fee and faucet behavior in some paths
- Additional serverless/admin/debug routes
- Does not mount the development multi-agent router

These directories share many concepts but are not fully behaviorally identical.
Changes to chain, RWA, transaction, or persistence behavior should be reviewed in
both places. Aligning them remains an important maintenance goal.

---

## Technology stack

| Layer | Technology |
|---|---|
| Frontend | React 18, TypeScript, Vite |
| Styling | CSS, JetBrains Mono, responsive terminal design |
| Backend | Node.js, Express, TypeScript |
| Persistence | SQLite via `better-sqlite3` |
| AI integrations | Anthropic Claude, OpenAI, Groq-compatible API |
| Contract source | Solidity |
| Testing/tooling | Jest, ts-jest, ESLint, Prettier, Artillery |
| Deployment configs | Replit workflow, Vercel, Railway, Docker |

---

## Project structure

```text
Vlad-Chain/
├── frontend/
│   ├── src/
│   │   ├── App.tsx                 # Main tab shell and chain UI
│   │   ├── RWADashboard.tsx        # RWA registry and analytics
│   │   ├── GIPSystem.tsx           # Governance proposal UI
│   │   ├── MultiAgentChat.tsx      # Validator chat UI
│   │   ├── LiveDebate.tsx          # Live debate presentation
│   │   ├── WalletConnect.tsx       # Wallet connection UI
│   │   └── index.css               # Global responsive terminal styles
│   └── public/
├── backend/
│   ├── src/
│   │   ├── index.ts                # Express application and core routes
│   │   ├── chain.ts                # In-memory chain simulation
│   │   ├── database.ts             # SQLite schema and persistence
│   │   ├── multi-agent.ts          # Canonical validator personas
│   │   ├── multi-agent-router.ts   # Multi-agent API
│   │   ├── personalities.ts        # Individual personality sessions
│   │   ├── rwa.ts                  # Synthetic RWA registry
│   │   ├── gip-system.ts           # Governance engine
│   │   └── gip-router.ts           # Governance API
│   └── data/                        # Local SQLite data (not committed)
├── api/                             # Vercel/serverless API variant
├── contracts/
│   └── VladToken.sol               # ERC-20 contract source
├── docs/
│   └── ARCHITECTURE.md
├── tests/
│   └── unit/
├── scripts/
│   └── push-to-github.mjs
├── Dockerfile
├── docker-compose.yml
├── railway.json
├── vercel.json
└── package.json
```

---

## Getting started

### Prerequisites

- Node.js 18 or newer
- npm 8 or newer

### Clone

```bash
git clone https://github.com/HeroDappDev/Vlad-Chain.git
cd Vlad-Chain
```

### Install

Install root, backend, and frontend dependencies:

```bash
npm run install:all
```

### Run the full development application

```bash
npm run dev
```

Default local addresses:

- Frontend: `http://localhost:5000`
- Backend: `http://localhost:4000`
- Health check: `http://localhost:4000/api/health`

### Run one side only

```bash
# Backend
npm run dev:backend

# Frontend
npm run dev:frontend
```

### Production-style build

```bash
npm run build
npm start
```

---

## Configuration

Environment variables are loaded by the relevant backend/provider modules.
Never commit real credentials.

| Variable | Purpose | Notes |
|---|---|---|
| `PORT` | Express server port | Defaults to `4000` |
| `CLAUDE_API_KEY` | Anthropic Claude access | Used by Claude/multi-agent paths |
| `OPENAI_API_KEY` | OpenAI access | Used by the OpenAI provider module |
| `GROK_API_KEY` | Groq-compatible API access | Used by the Groq provider module |
| `DATABASE_URL` | Database selection hint | PostgreSQL URLs currently fall back to SQLite |
| `DB_TYPE` | Database type hint | SQLite is the working implementation |

If no relevant AI provider key is configured, some AI interactions may return an
error or local fallback while the chain and RWA simulation continue working.

---

## Available scripts

Run these from the repository root:

| Script | Purpose |
|---|---|
| `npm run dev` | Run backend and frontend together |
| `npm run dev:backend` | Run only the Express development server |
| `npm run dev:frontend` | Run only the Vite frontend |
| `npm run build` | Build backend and frontend |
| `npm run build:backend` | Compile backend TypeScript |
| `npm run build:frontend` | Create the frontend production bundle |
| `npm start` | Start the compiled backend |
| `npm run test:unit` | Run existing Jest unit tests |
| `npm run test:watch` | Run Jest in watch mode |
| `npm run lint` | Run ESLint |
| `npm run lint:fix` | Run ESLint with automatic fixes |
| `npm run format` | Format supported files with Prettier |
| `npm run type-check` | Run root TypeScript checking |
| `npm run docker:build` | Build the Docker image |
| `npm run docker:run` | Run the image on port 4000 |
| `npm run docker:compose` | Start Docker Compose services |
| `npm run deploy:vercel` | Invoke Vercel CLI deployment |
| `npm run deploy:railway` | Invoke Railway CLI deployment |
| `npm run docs:generate` | Generate TypeDoc output |

The root `npm test` script references unit, integration, and end-to-end
directories. Only the existing test files should be treated as active coverage;
the other suites may require additional implementation.

---

## REST API reference

The routes below describe the long-running development backend unless noted
otherwise.

### Health and network

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/health` | Basic service health response |
| `GET` | `/api/stats` | Network totals and current chain statistics |
| `GET` | `/api/epoch` | Current epoch and slot |
| `POST` | `/api/advance_epoch` | Advance the simulated epoch |

### Blocks, accounts, and transactions

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/blocks` | Recent blocks |
| `GET` | `/api/all-blocks` | Full in-memory block list |
| `GET` | `/api/accounts` | Known accounts and balances |
| `GET` | `/api/validators` | Validator list and statistics |
| `GET` | `/api/pending` | Pending transaction pool |
| `GET` | `/api/transactions` | In-memory transaction history |
| `GET` | `/api/tx/:txId` | Persisted/in-memory transaction lookup |
| `POST` | `/api/create_account` | Add a simulation account |
| `POST` | `/api/generate_wallet` | Generate a simulation wallet |
| `POST` | `/api/faucet` | Request simulated test tokens |
| `POST` | `/api/send` | Submit a transfer and receive a receipt |
| `POST` | `/api/block` | Produce a block with a requested validator |
| `POST` | `/api/narrative` | Generate a human-readable transaction narrative |

#### Send request

```json
{
  "from": "sender-address",
  "to": "recipient-address",
  "amount": 25
}
```

#### Send response shape

```json
{
  "ok": true,
  "tx": {
    "hash": "generated-tx-id",
    "from": "sender-address",
    "to": "recipient-address",
    "amount": 25,
    "fee": 0.001,
    "timestamp": 1787990000000,
    "blockHeight": 123
  },
  "receipt": {
    "txId": "generated-tx-id",
    "blockHeight": 123,
    "timestampUTC": "2026-08-29T12:00:00.000Z",
    "from": "sender-address",
    "to": "recipient-address",
    "amount": 25,
    "fee": 0.001
  }
}
```

### RWA registry

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/rwa/registry` | All synthetic RWA assets |
| `GET` | `/api/rwa/stats` | Aggregated RWA metrics |
| `GET` | `/api/rwa/asset/:id` | Asset by internal ID or symbol |

Example:

```bash
curl http://localhost:4000/api/rwa/asset/vTBILL
```

### AI personalities and multi-agent chat

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/personality/:validator` | Send a message to a personality |
| `POST` | `/api/personality/clear-session` | Clear personality session state |
| `GET` | `/api/multi-agent/agents` | List available validator agents |
| `GET` | `/api/multi-agent/agent/:agentId` | Get one agent profile |
| `POST` | `/api/multi-agent/chat` | Run a multi-agent conversation |
| `POST` | `/api/multi-agent/chat/:agentId` | Chat with a selected agent |
| `POST` | `/api/multi-agent/chat/random` | Chat with a selected/random agent path |
| `POST` | `/api/multi-agent/simulate` | Run a simulated agent interaction |
| `GET` | `/api/multi-agent/history` | Read conversation history |
| `DELETE` | `/api/multi-agent/history` | Clear conversation histories |

The `/api/multi-agent` router is available in `backend/src` and is not mounted by
the serverless `api/index.ts` variant.

### Chat log

The `/api/chatlog` router supports reading, adding, and clearing terminal chat
records used by the activity feed.

### Governance

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/gip` | List proposals |
| `GET` | `/api/gip/active` | List active proposals |
| `GET` | `/api/gip/archived` | List archived proposals |
| `GET` | `/api/gip/:gipId` | Read one proposal |
| `GET` | `/api/gip/:gipId/transcript` | Read its debate transcript |
| `POST` | `/api/gip/:gipId/debate` | Start or advance debate |
| `POST` | `/api/gip/:gipId/archive` | Archive a proposal |
| `GET` | `/api/gip/stats/system` | Governance statistics |

For the exact current route definitions, use:

- `backend/src/index.ts`
- `backend/src/multi-agent-router.ts`
- `backend/src/personalities.ts`
- `backend/src/gip-router.ts`
- `backend/src/rwa.ts`

---

## Testing and validation

### Unit tests

```bash
npm run test:unit
```

The current repository contains chain-focused unit coverage under `tests/unit`.
The root scripts also define integration, end-to-end, performance, and benchmark
commands, but those should not be interpreted as complete coverage unless the
corresponding test files exist.

### Build validation

```bash
npm run build:backend
npm run build:frontend
```

### Manual smoke checks

After starting the app:

```bash
curl http://localhost:4000/api/health
curl http://localhost:4000/api/stats
curl http://localhost:4000/api/rwa/stats
```

Then verify the browser flows for Explorer, Faucet, Send, TX search, Oracle,
RWA, and GIPs.

---

## Deployment

The repository includes configuration for several deployment approaches:

- **Replit** — run the configured development workflow with `npm run dev`
- **Vercel** — uses the `api/` serverless variant and `vercel.json`
- **Railway** — configuration in `railway.json`
- **Docker** — configuration in `Dockerfile` and `docker-compose.yml`

See [`DEPLOYMENT.md`](./DEPLOYMENT.md) for the existing deployment notes.

Because `backend/src` and `api/` are not fully identical, deployment behavior
must be tested against the selected target rather than assumed from local
development.

---

## Security and limitations

This project must be treated as a simulation:

- Do not send real funds to any generated address.
- Do not reuse generated mnemonic-style phrases.
- Do not treat displayed RWA values as market data or investment information.
- Do not treat compliance labels as legal or regulatory approval.
- Do not treat AI responses as financial, legal, or security advice.
- Do not expect production-grade consensus, cryptographic signing, finality, or
  durable ledger guarantees.
- Some network state is held only in memory and resets with the process.
- CORS is broad in the development server.
- Operational mutation endpoints are intended for a demonstration environment.

For private vulnerability reporting, follow
[`SECURITY.md`](./SECURITY.md). Do not publish sensitive vulnerability details
in a public issue.

---

## Contributing

Contributions are welcome. Before opening a pull request:

1. Read [`CONTRIBUTING.md`](./CONTRIBUTING.md).
2. Keep the terminal design and mobile responsiveness intact.
3. Update both backend variants when behavior is expected to match.
4. Avoid documenting simulated features as real infrastructure.
5. Run the relevant build and test commands.
6. Update [`CHANGELOG.md`](./CHANGELOG.md) for user-visible changes.

---

## License

VLAD CHAIN is available under the [MIT License](./LICENSE).

Copyright © 2026 VladChain.