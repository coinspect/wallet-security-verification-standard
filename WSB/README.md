# Wallet Security Benchmark (WSB)

The WSB is a set of interaction-based security tests for web3 wallets, currently covering phishing protection and physical access scenarios. No source code access required.

## dApp Permissions (PERM)

Evaluates how the wallet manages dApp permissions and restricts sensitive RPCs. Includes enforcing user consent for connections, disclosing access scopes, and enforcing unlocks, plus tools to review or revoke persistent connections and token allowances.

**Attack scenario:** A malicious dApp exploits weak access controls, silent RPC execution, or undisclosed permission scopes to drain funds or perform unauthorized actions without explicit user consent.

| ID | Name | Description |
|---|---|---|
| [WSB-PERM-001](#wsb-perm-001) | Mismatching SIWE domain detection | Warns users when the domain in a Sign-In with Ethereum (EIP-4361) message does not match the requesting dApp's origin. |
| [WSB-PERM-002~Mobile](#wsb-perm-002mobile) | Wallet unlock before requests | Requires users to unlock it before processing dApp requests when in a locked state. |
| [WSB-PERM-002~Browser](#wsb-perm-002browser) | Wallet unlock before requests | Requires users to unlock it before processing dApp requests when in a locked state. |
| [WSB-PERM-003](#wsb-perm-003) | Mismatching EIP-712 chainId detection | Alerts users or rejects signing EIP-712 messages with a mismatched chain ID. |
| [WSB-PERM-004](#wsb-perm-004) | Connected dApp management | Allows users to list and revoke connected dApps. |
| [WSB-PERM-005](#wsb-perm-005) | Token approval management | Allows users to view and revoke token approvals. |
| [WSB-PERM-006](#wsb-perm-006) | eth_sign method disabled | Restricts the use of the deprecated and insecure `eth_sign` method by default. |
| [WSB-PERM-007~Mobile](#wsb-perm-007mobile) | Confirmation for requests from WalletConnect | Requires user confirmation for requests from WalletConnect before granting dApp access to specific RPC methods. |
| [WSB-PERM-008~Browser](#wsb-perm-008browser) | User consent for dApp access | Requires user approval before granting dApp access to specific RPC methods. |
| [WSB-PERM-009](#wsb-perm-009) | User confirmation before switching chains | Requires user confirmation before switching the active chain. |
| [WSB-PERM-010~Mobile](#wsb-perm-010mobile) | User confirmation for requests from embedded browser | Requires user confirmation before processing privileged RPC requests from embedded browsers. |
| [WSB-PERM-010~Browser](#wsb-perm-010browser) | User confirmation before processing requests | Requires user confirmation before processing requests from dApps for specific RPC methods. |

---

### WSB-PERM-001

**Mismatching SIWE domain detection**

Warns users when the domain in a Sign-In with Ethereum (EIP-4361) message does not match the requesting dApp's origin.

**Attack scenario:** A phishing site tricks the user into signing a login message that authenticates them to a different service targeted by the attacker.

**Testing instructions:** Generate and attempt to sign a SIWE message from a dApp whose origin differs from the domain specified in the SIWE object. Observe whether the wallet warns the user or blocks the signing.

**Reasoning:** A mismatch between the dApp origin and the domain inside the SIWE message can indicate phishing or session hijacking attempt. Warning on this helps users avoid signing login messages that could authenticate them to a different service than the one they think they are using.

---

### WSB-PERM-002~Mobile

**Wallet unlock before requests** · Mobile

Requires users to unlock it before processing dApp requests when in a locked state.

**Attack scenario:** An attacker with temporary access to the device triggers wallet operations or extracts sensitive data while the wallet is locked.

**Testing instructions:** If the wallet implements WalletConnect, with the wallet previously connected but in a locked state, send requests from a dApp to the following RPC endpoints: `wallet_addEthereumChain`, `wallet_watchAsset`, `eth_decrypt`, `eth_getEncryptionPublicKey`, `eth_signTypedData*`, `personal_sign`, `eth_sendTransaction`. Observe whether each request triggers an authentication prompt before being processed and look for possible information leaks.

**Reasoning:** A locked wallet must never continue serving sensitive requests, since that would weaken the purpose of the lock state. Requiring an unlock prevents unauthorized access to accounts, signatures, or encrypted material if the device is unattended or temporarily exposed.

---

### WSB-PERM-002~Browser

**Wallet unlock before requests** · Browser

Requires users to unlock it before processing dApp requests when in a locked state.

**Attack scenario:** An attacker with temporary access to the device triggers wallet operations or extracts sensitive data while the wallet is locked.

**Testing instructions:** With the wallet already connected to a dApp but in a locked state, send requests from the dApp to each of the following RPC endpoints: `wallet_addEthereumChain`, `wallet_watchAsset`, `eth_decrypt`, `eth_getEncryptionPublicKey`, `eth_signTypedData*`, `personal_sign`, and `eth_sendTransaction`. Observe whether each request triggers an authentication prompt before being processed and look for possible information leaks.

**Reasoning:** A locked wallet must never continue serving sensitive requests, since that would weaken the purpose of the lock state. Requiring an unlock prevents unauthorized access to accounts, signatures, or encrypted material if the device is unattended or temporarily exposed.

---

### WSB-PERM-003

**Mismatching EIP-712 chainId detection**

Alerts users or rejects signing EIP-712 messages with a mismatched chain ID.

**Attack scenario:** A malicious dApp tricks the user into signing data for a different network than expected, enabling misuse of the signature under a misleading execution context.

**Testing instructions:** Craft an EIP-712 object with a `chainId` different from the wallet's currently active network and attempt to sign it. For example, if the wallet is connected to Polygon (`chainId` 137), attempt signing an EIP-712 object with a different `chainId` and observe whether the wallet warns or rejects the request.

**Reasoning:** EIP-712 signatures are domain-separated partly through the `chainId`, so signing data for a different chain can enable cross-chain confusion or replay-like abuse in poorly designed integrations. Warning or rejecting mismatched `chainId`s helps users understand what environment they are authorizing and prevents signatures from being produced under misleading assumptions.

---

### WSB-PERM-004

**Connected dApp management**

Allows users to list and revoke connected dApps.

**Attack scenario:** A previously trusted dApp becomes malicious or is compromised, and due to another vulnerability or excessive permissions, it can abuse its existing connection to perform unintended actions or access sensitive data.

**Testing instructions:** Having previously connected to at least two dApps, look for a section within the wallet UI that lists all connected dApps. Disconnect the testing dApp, then send RPC requests from it to verify access has been fully revoked.

**Reasoning:** Persistent dApp connections represent an ongoing trust relationship, so users need visibility into which apps still have access. The ability to review and revoke those connections reduces the risk of stale approvals, forgotten sessions, or continued access after the user no longer trusts the dApp.

---

### WSB-PERM-005

**Token approval management**

Allows users to view and revoke token approvals.

**Attack scenario:** A malicious or compromised contract or address uses previously granted token approvals to transfer tokens without additional user consent.

**Testing instructions:** Look for functionality within the wallet UI to list and revoke token approvals, either as a built-in feature or as a link to an external dApp.

**Reasoning:** Token approvals can outlive a single transaction and are a common source of loss when a spender contract is compromised or malicious. Letting users inspect and revoke approvals gives them a practical way to reduce standing risk and clean up excessive allowances.

---

### WSB-PERM-006

**eth_sign method disabled**

Restricts the use of the deprecated and insecure `eth_sign` method by default.

**Attack scenario:** A malicious dApp tricks the user into signing opaque data that can be interpreted as a valid transaction or authorization.

**Testing instructions:** With the wallet connected and unlocked, send an `eth_sign` RPC request via a dApp and observe the response. Note that some wallets silently redirect `eth_sign` calls to `personal_sign` instead.

**Reasoning:** `eth_sign` is historically dangerous because it signs raw data in a way that is easy to misuse and hard for users to interpret safely. In particular, it can be abused to trick users into signing raw transaction hashes or other opaque payloads without clear context. Disabling it by default reduces compatibility with insecure legacy flows and pushes dApps toward safer, more explicit signing methods.

---

### WSB-PERM-007~Mobile

**Confirmation for requests from WalletConnect** · Mobile

Requires user confirmation for requests from WalletConnect before granting dApp access to specific RPC methods.

**Attack scenario:** A malicious dApp triggers sensitive operations through WalletConnect without clear user awareness or intent.

**Testing instructions:** If the wallet implements WalletConnect, connect to a dApp via WalletConnect and send requests to the following RPC endpoints: `wallet_addEthereumChain`, `wallet_watchAsset`, `eth_decrypt`, `eth_getEncryptionPublicKey`, `eth_signTypedData*`, `personal_sign`, `eth_sendTransaction`. Observe whether each request surfaces a confirmation prompt.

**Reasoning:** Requiring confirmation for sensitive RPC methods ensures the dApp cannot silently trigger signing, encryption, asset watching, or transaction actions without the user noticing.

---

### WSB-PERM-008~Browser

**User consent for dApp access** · Browser

Requires user approval before granting dApp access to specific RPC methods.

**Attack scenario:** A malicious or untrusted dApp accesses accounts or initiates sensitive requests without explicit user authorization.

**Testing instructions:** Without first connecting the wallet to a dApp, send requests to the following RPC endpoints: `eth_accounts`, `eth_decrypt`, `eth_getEncryptionPublicKey`, `eth_signTypedData*`, `personal_sign`, `eth_sendTransaction`. Observe whether each request triggers a connection prompt or is rejected, and there are no information leaks.

**Reasoning:** A dApp should not be able to access accounts or request sensitive operations before the user explicitly grants it permission. Enforcing a connection step prevents passive data exposure and makes the trust decision visible and deliberate.

---

### WSB-PERM-009

**User confirmation before switching chains**

Requires user confirmation before switching the active chain.

**Attack scenario:** A malicious dApp switches the user to an unexpected network where assets or interactions behave differently, leading to unintended actions.

**Testing instructions:** Send a `wallet_switchEthereumChain` RPC request from a dApp and observe whether a confirmation prompt appears. If no prompt is shown, check whether the wallet allows users to manage pre-approved networks within the connection dialog — if so, configure only one approved network and attempt to switch to any other chain. Perform this test through both the browser interface and the WalletConnect interface, if applicable.

**Reasoning:** Chain switching changes the execution context of future transactions and signatures, so it should never happen silently. Confirmation protects users from being moved onto an unexpected network where balances, assets, or contract interactions may behave very differently.

---

### WSB-PERM-010~Mobile

**User confirmation for requests from embedded browser** · Mobile

Requires user confirmation before processing privileged RPC requests from embedded browsers.

**Attack scenario:** A malicious dApp triggers privileged operations from an embedded browser without the user clearly reviewing and approving each request.

**Testing instructions:** If the wallet has an embedded browser, it requires user confirmation before processing each dApp request to the following RPC endpoints: `wallet_addEthereumChain`, `wallet_watchAsset`, `eth_decrypt`, `eth_getEncryptionPublicKey`, `eth_signTypedData*`, `personal_sign`, `eth_sendTransaction`.

**Reasoning:** Requiring confirmation for sensitive RPC methods ensures the dApp cannot silently trigger signing, encryption, asset watching, or transaction actions without the user noticing.

---

### WSB-PERM-010~Browser

**User confirmation before processing requests** · Browser

Requires user confirmation before processing requests from dApps for specific RPC methods.

**Attack scenario:** A malicious dApp executes privileged operations (signing, transactions, approvals) without the user explicitly reviewing and approving each request.

**Testing instructions:** Send requests to the following RPC endpoints: `wallet_addEthereumChain`, `wallet_watchAsset`, `eth_decrypt`, `eth_getEncryptionPublicKey`, `eth_signTypedData*`, `personal_sign`, `eth_sendTransaction`. Observe whether each requires user confirmation.

**Reasoning:** Sensitive RPC methods can lead to asset movement, approval of malicious contracts, disclosure of encryption material, or misleading signatures. Requiring per-request confirmation ensures the user reviews each action in context instead of the dApp operating with broad invisible permission.

---

## Intent Verification (VERI)

Evaluates how effectively the wallet decodes and presents transaction data. Includes translating raw hex/EIP-712 into readable formats, simulating outcomes, validating checksums, and ensuring payloads are fully visible and reviewed before signing.

**Attack scenario:** An attacker tricks the user into blind-signing opaque or partially hidden payloads, or exploits mistyped addresses, causing unintended asset transfers to malicious destinations.

| ID | Name | Description |
|---|---|---|
| [WSB-VERI-001](#wsb-veri-001) | Invalid address checksum detection | Warns users when they input addresses with invalid EIP-55 checksums. |
| [WSB-VERI-002](#wsb-veri-002) | Links to blockchain explorers | Consistently provides clickable links to block explorers for all key blockchain identifiers. |
| [WSB-VERI-003](#wsb-veri-003) | Transaction simulation | Previews the expected outcome by simulating the request execution on the blockchain before signing. |
| [WSB-VERI-004](#wsb-veri-004) | EIP-712 message parsing | Displays human-readable details for EIP-712 signature requests from well-known protocols. |
| [WSB-VERI-005](#wsb-veri-005) | Clear token approval dialog | Clearly displays all the key details for ERC-20 Approve requests. |
| [WSB-VERI-006](#wsb-veri-006) | Clear message signing dialog | Clearly displays all message signature request details without truncating or hiding information. |
| [WSB-VERI-007](#wsb-veri-007) | Mandatory message review | Requires users to review all the details before signing a message. |

---

### WSB-VERI-001

**Invalid address checksum detection**

Warns users when they input addresses with invalid EIP-55 checksums.

**Attack scenario:** A user sends funds to a mistyped or manipulated address due to input errors or clipboard tampering.

**Testing instructions:** Attempt to send a transaction to an address with an incorrect EIP-55 checksum (e.g. alter the case of any letter). The transaction should be attempted manually and from a dApp. Observe whether the wallet surfaces a warning or blocks the transaction.

**Reasoning:** EIP-55 checksums are meant to catch address-entry mistakes that would otherwise be hard to spot. Warning on an invalid checksum reduces the chance of sending funds to a mistyped or malformed address, especially when users copy addresses from unreliable sources.

---

### WSB-VERI-002

**Links to blockchain explorers**

Consistently provides clickable links to reputable explorers for all key blockchain identifiers.

**Attack scenario:** A malicious interface misrepresents transaction or address details, preventing independent verification by the user.

**Testing instructions:** Attempt to send a transaction and inspect the transaction details screen. Also browse the wallet's transaction history. Observe whether addresses, contract addresses, and transaction hashes are displayed as clickable links to a block explorer.

**Reasoning:** Explorer links let users independently verify addresses, contracts, and transactions instead of trusting only the wallet UI. This improves transparency and makes it easier to inspect counterparties, token contracts, and transaction outcomes before or after signing.

---

### WSB-VERI-003

**Transaction simulation**

Previews the expected outcome by simulating the request execution on the blockchain before signing.

**Attack scenario:** A malicious dApp disguises the real effects of a transaction, such as draining funds or granting approvals, until after it is signed.

**Testing instructions:** Attempt to provide liquidity into a liquidity pool and perform a swap (e.g. Uniswap) and observe whether the wallet shows the expected inputs and outputs from the execution of such transaction before it is signed.`

**Reasoning:** Many dangerous transactions only become obvious when looking at their actual effects, not just calldata or method names. Simulation helps users see expected token movements, approvals, or state changes in advance, which is one of the strongest defenses against deceptive dApp flows.

---

### WSB-VERI-004

**EIP-712 message parsing**

Displays human-readable details for EIP-712 signature requests from well-known protocols.

**Attack scenario:** A malicious dApp crafts structured data that appears benign but actually grants permissions or authorizations when signed.

**Testing instructions:** Trigger EIP-712 signature requests from well-known protocols such as OpenSea Seaport, ERC-20 Permit flows, or Uniswap, and observe how the wallet renders them — whether it shows structured, human-readable detail or raw JSON.

**Reasoning:** EIP-712 is designed to make structured data more understandable, but that only works if the wallet parses and presents it clearly. Human-readable rendering helps users distinguish between harmless messages and signatures that authorize critical permissions or off-chain actions.

---

### WSB-VERI-005

**Clear token approval dialog**

Clearly displays all the key details for ERC-20 Approve requests.

**Attack scenario:** A malicious dApp tricks the user into approving excessive or unlimited token allowances to an attacker-controlled contract.

**Testing instructions:** Trigger an ERC-20 approve in two ways: 1) deploy a custom ERC-20 and call approve directly, and 2) access any swap dApp and trigger an approve through the UI. Observe what information the wallet displays about the token, spender, and amount.

**Reasoning:** ERC-20 approvals can grant a spender long-lived control over tokens. Clearly showing the token, spender, and amount helps users detect malicious approvals, unlimited allowances, and approvals targeting unexpected contracts.

---

### WSB-VERI-006

**Clear message signing dialog**

Clearly displays all message signature request details without truncating or hiding information.

**Attack scenario:** A malicious dApp hides or obfuscates critical parts of a signing payload, leading the user to approve unintended or harmful data.

**Testing instructions:** Send large payloads via `personal_sign` and `eth_signTypedData` and observe how the wallet displays the data — whether all content is visible, scrollable, and not truncated. For EIP-712 objects, also check whether the `EIP712Domain`, including the `verifyingContract` address, is displayed.

**Reasoning:** Users cannot give meaningful consent if important parts of the message are hidden, truncated, or impossible to inspect. Full visibility into signed data is critical because attackers can bury harmful payloads or structured objects.

---

### WSB-VERI-007

**Mandatory message review**

Requires users to review all the details before signing a message.

**Attack scenario:** A malicious dApp relies on users blindly approving signature requests without reviewing their contents, leading to unintended authorizations.

**Testing instructions:** Send a large payload via `personal_sign` or `eth_signTypedData` and observe whether the sign button is accessible before scrolling to the end of the content.

**Reasoning:** Forcing the user to scroll through the full request makes rushed or accidental approvals less likely. It is not a perfect defense, but it raises friction in exactly the places where attackers benefit from users clicking through without reviewing the content.

---

## Threat Prevention (THRE)

Evaluates proactive measures against deceptive dApps and external threats. Includes blocklists for phishing and malicious contracts, identification of trusted domains, and filtering out malicious tokens or NFTs.

**Attack scenario:** A user interacts with a phishing site or malicious contract, and the wallet fails to intercept the interaction or warn about the risk, leading to direct asset loss.

| ID | Name | Description |
|---|---|---|
| [WSB-THRE-001](#wsb-thre-001) | Trusted dApp detection | Informs users when interacting with a trusted dApp URL. |
| [WSB-THRE-002](#wsb-thre-002) | Malicious address detection | Prevents or alerts users about interactions with a known malicious address. |
| [WSB-THRE-003](#wsb-thre-003) | Phishing dApp detection | Prevents or alerts users about interactions with a known malicious URL. |
| [WSB-THRE-004](#wsb-thre-004) | dApp access disclosure dialog | Informs dApp access to balances, history, and signing requests on connection. |
| [WSB-THRE-005](#wsb-thre-005) | Unknown address detection | Warns users when interacting with an unknown address. |
| [WSB-THRE-006](#wsb-thre-006) | Full dApp URL display | Clearly displays the full dApp URL in the connection prompt. |
| [WSB-THRE-007](#wsb-thre-007) | Malicious or spam token filtering | Hides malicious tokens and NFTs by default. |

---

### WSB-THRE-001

**Trusted dApp detection**

Informs users when interacting with a trusted dApp URL.

**Attack scenario:** A phishing site impersonates a trusted dApp to gain user trust and request sensitive actions.

**Testing instructions:** Attempt to connect to a well-known dApp (e.g. Uniswap) and observe the connection request screen. Then repeat with an unknown dApp and compare how each is displayed.

**Reasoning:** Verified dApp indicators help users distinguish trusted, well-known services from lookalike phishing sites. This is especially valuable during connection prompts, where users are often making a quick trust decision with limited information.

---

### WSB-THRE-002

**Malicious address detection**

Prevents or alerts users about interactions with a known malicious address.

**Attack scenario:** A user unknowingly sends funds to an address associated with scams, theft, or malicious activity.

**Testing instructions:** Attempt to send a transaction to a known malicious address (e.g. the Tornado Cash attacker address). Test both via dApp and direct wallet UI. Observe whether the wallet surfaces any warning.

**Reasoning:** Many scams rely on users sending funds to addresses already known to be linked to theft, laundering, or fraud. Alerting on such addresses gives users a last defensive checkpoint before irreversible on-chain transfers.

---

### WSB-THRE-003

**Phishing dApp detection**

Prevents or alerts users about interactions with a known malicious URL.

**Attack scenario:** A user interacts with a known malicious website that attempts to steal funds or signatures.

**Testing instructions:** Visit or attempt to connect to sites included in the [MetaMask phishing list](https://github.com/MetaMask/eth-phishing-detect/blob/b68a72cc477273db3717204d5db699c11980b2f5/src/config.json) and observe whether the wallet warns the user. Note that Chrome also includes built-in phishing protection, which may trigger warnings for some of these sites, though not all.

**Reasoning:** Phishing websites are one of the most common ways users lose funds or sign malicious payloads. Detecting and warning on known phishing domains blocks attacks early, before the user even reaches the approval stage.

---

### WSB-THRE-004

**dApp access disclosure dialog**

Informs dApp access to balances, history, and signing requests on connection.

**Attack scenario:** A user grants access to a malicious dApp without understanding the scope of its capabilities, allowing it to later exploit its permissions or any wallet-side vulnerabilities once connected.

**Testing instructions:** Attempt connecting a dApp to the wallet and inspect the connection dialog for any description of what access the dApp is being granted.

**Reasoning:** Explaining the scope of access during the connection flow improves informed consent and reduces the chance that users treat connection prompts as harmless.

---

### WSB-THRE-005

**Unknown address detection**

Warns users when interacting with an unknown address.

**Attack scenario:** A user sends funds to an unfamiliar or unverified address without recognizing the associated risk.

**Testing instructions:** Send a transaction to a fresh address — one the account has never interacted with and that is not saved as a contact — both through a dApp and directly through the wallet UI. Observe whether any warning or indicator appears.

**Reasoning:** Highlighting unknown recipients creates a useful pause before sending funds or interacting with a new contract that may be malicious or mistyped.

---

### WSB-THRE-006

**Full dApp URL display**

Clearly displays the full dApp URL in the connection prompt.

**Attack scenario:** A phishing site uses deceptive or truncated URLs to impersonate a legitimate service.

**Testing instructions:** Attempt to connect the wallet to a dApp and observe how the origin URL is displayed in the connection dialog.

**Reasoning:** The origin URL is one of the most important signals for spotting phishing or impersonation attempts. Showing the full, non-truncated URL helps users notice deceptive subdomains, typosquatting, or other visual tricks that could be hidden by truncation.

---

### WSB-THRE-007

**Spam token filtering**

Filters unsolicited tokens by default to protect users from potentially malicious interactions.

**Attack scenario:** Spam tokens or NFTs trick users into interacting with malicious links, contracts, or social engineering schemes.

**Testing instructions:** Using a test wallet that has received spam NFTs and tokens, open the wallet and observe which assets are displayed by default.

**Reasoning:** Spam tokens and NFTs are commonly used for phishing, impersonation, and social engineering inside wallet portfolios. Hiding them by default reduces clutter and lowers the chance that users click malicious assets or mistake them for legitimate holdings.

---

## Physical Access (PHYS)

Evaluates defenses against unauthorized physical access and local data exposure. Includes strong authentication, auto-locking, rate limiting, and restricting seed phrase exposure via clipboards, screenshots, or unauthenticated views.

**Attack scenario:** An attacker with temporary physical access exploits an unlocked session or weak local authentication to authorize transactions or exfiltrate secrets (like the seed phrase).

| ID | Name | Description |
|---|---|---|
| [WSB-PHYS-001](#wsb-phys-001) | Clipboard seed phrase leak prevention | Limits exposure of secrets by restricting or warning on copying seed phrases or taking screenshots. |
| [WSB-PHYS-002~Mobile](#wsb-phys-002mobile) | Robust authentication | Uses strong authentication, including biometrics, rate limiting, and resistance to trivial credentials. |
| [WSB-PHYS-002~Browser](#wsb-phys-002browser) | Robust authentication | Uses strong authentication, such as resistance to trivial or commonly-used passwords. |
| [WSB-PHYS-003](#wsb-phys-003) | Seed phrase access warning | Warns users of the risk before allowing access to seed phrases or private keys. |
| [WSB-PHYS-004](#wsb-phys-004) | Manual wallet lock | Allows users to lock it manually. |
| [WSB-PHYS-005](#wsb-phys-005) | Seed phrase access control | Requires authentication to access seed phrases or private keys. |
| [WSB-PHYS-006~Mobile](#wsb-phys-006mobile) | Automatic wallet lock | Verifies wallet auto-locks on inactivity, device lock, or background. |
| [WSB-PHYS-006~Browser](#wsb-phys-006browser) | Automatic wallet lock | Verifies wallet auto-locks on inactivity. |

---

### WSB-PHYS-001

**Clipboard seed phrase leak prevention**

Limits exposure of secrets by restricting or warning on copying seed phrases or taking screenshots.

**Attack scenario:** Seed phrases are exposed through clipboard access or screenshots that may be stored locally, synced to the cloud, or accessed by someone with physical access to the device.

**Testing instructions:** Navigate to the seed phrase screen and copy the mnemonic to the clipboard. Observe whether the wallet clears it after a short time or displays a warning about clipboard risks. On mobile, also attempt to take a screenshot of the seed phrase screen and observe the wallet's response.

**Reasoning:** Seed phrases are extremely sensitive, and clipboards and screenshots are common exfiltration paths on both desktop and mobile systems. Limiting, warning, or blocking these actions reduces the exposure window and makes users think twice before handling secrets insecurely.

---

### WSB-PHYS-002~Mobile

**Robust authentication**

Uses strong authentication, including biometrics, rate limiting, and resistance to trivial credentials.

**Attack scenario:** An attacker with access to the device bypasses weak authentication mechanisms (e.g., weak passwords, lack of rate limiting) to gain control of the wallet.

**Testing instructions:** Check whether biometric authentication is available and attempt to set a weak PIN (e.g. `1234`). Then attempt multiple incorrect logins and observe whether rate limiting activates.

**Reasoning:** Weak local authentication can completely undermine all other security controls. Strong passwords, biometrics, and rate-limiting make brute force, shoulder surfing, and opportunistic device access much harder.

---
### WSB-PHYS-002~Browser

**Robust authentication**

Uses strong authentication, such as resistance to trivial or commonly-used passwords.

**Attack scenario:** An attacker with access to the device bypasses weak authentication mechanisms (e.g., weak passwords) to gain control of the wallet.

**Testing instructions:** Attempt short or trivial passwords such as `1234`, `12345678`, `qwertyui`, or `password` and observe whether they are accepted.

**Reasoning:** Weak local authentication can completely undermine all other security controls. Strong passwords, biometrics, and rate-limiting make brute force, shoulder surfing, and opportunistic device access much harder.

---

### WSB-PHYS-003

**Seed phrase access warning**

Warns users of the risk before allowing access to seed phrases or private keys.

**Attack scenario:** A user exposes their seed phrase without understanding it grants full control over their funds.

**Testing instructions:** Navigate to the seed phrase or private key reveal screen and observe whether the risks of sharing the secrets are shown.

**Reasoning:** Users sometimes reveal seed phrases or private keys without fully understanding that doing so grants complete control over their assets. A clear warning at the moment of exposure can prevent catastrophic mistakes and reinforces the severity of the action.

---

### WSB-PHYS-004

**Manual wallet lock**

Allows users to lock it manually.

**Attack scenario:** An attacker with access to the device interacts with an unlocked wallet session to perform unauthorized actions.

**Testing instructions:** Look for a manual lock button within the wallet UI.

**Reasoning:** Users need a fast way to secure the wallet when stepping away, sharing a screen, or using a device in public. Manual locking gives immediate control rather than forcing the user to wait for an inactivity timer.

---

### WSB-PHYS-005

**Seed phrase access control**

Requires authentication to access seed phrases or private keys.

**Attack scenario:** An attacker with access to the device retrieves seed phrases or private keys due to missing or weak re-authentication controls.

**Testing instructions:** Navigate to the seed phrase or private key display screen and observe whether authentication is required before the secrets are shown.

**Reasoning:** Seed phrases and private keys should be protected behind strong re-authentication because they are the ultimate account recovery and control mechanism. Displaying them without an authentication step makes accidental exposure and local compromise far more likely.

---

### WSB-PHYS-006~Mobile

**Automatic wallet lock** · Mobile

The wallet auto-locks on inactivity, device lock, or background.

**Attack scenario:** An attacker with access to the device exploits an idle or background wallet session that remains unlocked.

**Testing instructions:** Review the auto-lock timer in the wallet settings. If it's set to 0, the wallet won't likely auto-lock. If the wallet does not expose an auto-lock setting, exit the wallet or leave it idle and observe whether it locks after a period of inactivity. Also verify whether the wallet also locks when the application is moved to the background or when the device itself is locked. 

**Reasoning:** Mobile devices are frequently lost, borrowed, or left unlocked briefly, so short auto-lock periods materially reduce the chance of unauthorized access.

---

### WSB-PHYS-006~Browser

**Automatic wallet lock** · Browser

The wallet auto-locks after a period of inactivity.

**Attack scenario:** An attacker with access to the device exploits an idle or unattended wallet session that remains unlocked.

**Testing instructions:** Check the wallet settings for an auto-lock configuration. If no setting is visible, leave the wallet idle for up to 20 minutes to observe whether it auto-locks. If it's 0 by default, then the wallet does not auto-lock on inactivity.

**Reasoning:** Browser extension wallets often remain open for long periods on shared or unattended computers. Auto-locking after a period of inactivity time reduces the window in which an attacker, coworker, or malicious webpage can exploit an already-unlocked wallet.
