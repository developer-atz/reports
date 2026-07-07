# Web3 Feature Review — Summary

## Overall status

The Web3 features (wallet connection, wallet login, send token) are working and several parts are well built: accurate gas fee estimation, safe transaction-confirmation waiting, and clear error messages for users. That said, there are a few security risks and quality gaps that should be planned for before scaling up the user base.

## Existing Web3 features

| # | Feature | Short description | Main improvement proposal |
|---|---|---|---|
| 1 | **Wallet connection** | Users connect via MetaMask, WalletConnect (QR scan), or Coinbase Wallet. | Consolidate the handling logic for the 3 wallet types (currently written separately, prone to inconsistent behavior between wallets); remove the "Brave Wallet" option, which doesn't actually work yet. |
| 2 | **Wallet login (SIWE)** | Authenticates the user's identity by signing a message instead of using a password. | **Top priority**: review the "signature reuse" protection mechanism together with the backend team. |
| 3 | **Send token** | A 5-step flow: choose recipient → choose currency/amount → confirm → sign → wait for blockchain confirmation. Automatically estimates gas fees and warns if the wallet lacks funds to cover them. | Add an instant warning when the user enters an incorrectly formatted recipient address; preserve transaction state if the user accidentally refreshes mid-flow. |
| 4 | **Multi-wallet management** | Users can link multiple wallets to one account, view the list, and switch the active wallet. | Add the ability to permanently remove a wallet from the account (currently only a temporary disconnect exists). |

*(Automated testing — which doesn't exist yet — is listed under "Priority risks" below, since it's a gap rather than a currently running feature.)*

## Flow diagram & improvement proposals

```text
┌──────────────────────────────────────┐
│ 1. CONNECT WALLET                    │
│    (MetaMask / WalletConnect / CB)   │
└──────────────────────────────────────┘
     • Consolidate 3 connector logic ........ [MEDIUM]
     • Move RPC API key to env var .......... [MEDIUM]
                    │
                    ▼
┌──────────────────────────────────────┐
│ 2. SIGN-IN (SIWE)                    │
└──────────────────────────────────────┘
     • Move nonce to server-issued .......... [CRITICAL]
     • Remove/gate debug page from prod ..... [HIGH]
     • Re-enable chain check before signing . [MEDIUM]
                    │
                    ▼
┌──────────────────────────────────────┐
│ 3. SESSION & MULTI-WALLET            │
└──────────────────────────────────────┘
     • Add permanent wallet-unlink feature .. [LOW]
                    │
                    ▼
┌──────────────────────────────────────┐
│ 4. SEND TOKEN                        │
│    Recipient → Amount → Confirm&Sign │
│    → Wait Confirmation               │
└──────────────────────────────────────┘
     [Recipient step]
     • Validate recipient address format .... [HIGH]
     • Warn on special addresses ............ [MEDIUM]
     [Amount step]
     • Proposal: split "choose token" into its
       own step, before "enter amount" ....... [LOW]
     [Wait Confirmation step]
     • Persist transaction state ............ [MEDIUM]
     • Separate RPC error from "failed" ..... [LOW]

     Cross-cutting:
     • Add automated tests .................. [MEDIUM]
```

Summary by step:

| Flow step | Improvement proposal | Priority |
|---|---|---|
| 1. Connect Wallet | Consolidate the handling logic for the 3 wallet types to avoid inconsistent behavior | Medium |
| 1. Connect Wallet | Move the RPC API key into an environment variable | Medium |
| 2. Sign-In (SIWE) | Move the nonce to server-issued to block signature-reuse risk | **Critical** |
| 2. Sign-In (SIWE) | Remove/gate the internal debug page from production | High |
| 2. Sign-In (SIWE) | Re-enable the chain check before signing | Medium |
| 3. Session & Multi-wallet | Add a permanent wallet-unlink feature | Low |
| 4. Send Token — Recipient step | Validate address format as soon as it's entered | High |
| 4. Send Token — Recipient step | Warn when sending to a special address (zero-address, token contract, self) | Medium |
| 4. Send Token — Amount step | Proposal: split "choose token" into its own step, before "enter amount" | Low |
| 4. Send Token — waiting-for-confirmation step | Persist transaction state so tracking isn't lost on refresh | Medium |
| 4. Send Token — waiting-for-confirmation step | Separate temporary network errors from "transaction failed" | Low |
| Cross-cutting | Add automated tests for money-sensitive flows | Medium |