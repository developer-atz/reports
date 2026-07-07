# Web3 Feature Review — Summary

## Overall status

The Web3 features (wallet connection, wallet login, send token) are working and several parts are well built: accurate gas fee estimation, safe transaction-confirmation waiting, and clear error messages for users. That said, there are a few security risks and quality gaps that should be planned for before scaling up the user base.

## Existing Web3 features

| # | Feature | Short description | Main improvement proposal |
|---|---|---|---|
| 1 | **Wallet connection** | Users connect via MetaMask, WalletConnect (QR scan), or Coinbase Wallet. | On mobile, connecting via deep link doesn't work — tapping to open the wallet app fails to complete the connection, so mobile users are effectively stuck. Also, disconnecting a Coinbase Wallet shows the loading spinner twice before it actually finishes. |
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
     • Fix mobile deep-link connect ......... [HIGH]
     • Coinbase disconnect double-loading ... [MEDIUM]
                    │
                    ▼
┌──────────────────────────────────────┐
│ 2. SIGN-IN (SIWE)                    │
└──────────────────────────────────────┘
     • Move nonce to server-issued .......... [CRITICAL]
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
| 1. Connect Wallet | Fix mobile deep-link connect, which currently fails to complete | High |
| 1. Connect Wallet | Coinbase Wallet disconnect shows the loading spinner twice before finishing | Medium |
| 2. Sign-In (SIWE) | Move the nonce to server-issued to block signature-reuse risk | **Critical** |
| 3. Session & Multi-wallet | Add a permanent wallet-unlink feature | Low |
| 4. Send Token — Recipient step | Validate address format as soon as it's entered | High |
| 4. Send Token — Recipient step | Warn when sending to a special address (zero-address, token contract, self) | Medium |
| 4. Send Token — Amount step | Proposal: split "choose token" into its own step, before "enter amount" | Low |
| 4. Send Token — waiting-for-confirmation step | Persist transaction state so tracking isn't lost on refresh | Medium |
| 4. Send Token — waiting-for-confirmation step | Separate temporary network errors from "transaction failed" | Low |
| Cross-cutting | Add automated tests for money-sensitive flows | Medium |