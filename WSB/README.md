# Wallet Security Benchmark (WSB)

The WSB is a set of black-box tests that can be performed against a wallet without access to its source code. The tests focus primarily on phishing protection — evaluating how well a wallet defends users against deceptive DApps, malicious signatures, and social engineering at the UI level.

## DApp Permissions (PERM)

Tests to ensure the wallet always asks before dApps access your balance or suggest transaction, and provides permissions control features such as token approvals management.

| ID | Name | Description |
|---|---|---|
| [WSB-PERM-001](#wsb-perm-001) | SIWE domain mismatch warning | Alerts users of discrepancies or unusual patterns in Sign-in with Ethereum (EIP-4361) requests. |
| [WSB-PERM-002~Mobile](#wsb-perm-002mobile) | Wallet unlock before requests | Requires users to unlock before processing DApp requests when in a locked state. |
| [WSB-PERM-002~Browser](#wsb-perm-002browser) | Wallet unlock before requests | Requires users to unlock before processing DApp requests when in a locked state. |
| [WSB-PERM-003](#wsb-perm-003) | Different ChainID | Alerts users or rejects requests to sign EIP-712 data with a chain ID different from the active chain. |
| [WSB-PERM-004](#wsb-perm-004) | Connected DApps | Allows users to list and revoke connected DApps. |
| [WSB-PERM-005](#wsb-perm-005) | List and revoke token approvals | Allows users to view and revoke token approvals. |
| [WSB-PERM-006](#wsb-perm-006) | Disables `eth_sign` method | Restricts the use of the deprecated and insecure `eth_sign` method by default. |
| [WSB-PERM-007~Mobile](#wsb-perm-007mobile) | Confirmation for WalletConnect requests | Requires user confirmation for WalletConnect requests before granting DApp access to specific RPC methods. |
| [WSB-PERM-008~Browser](#wsb-perm-008browser) | Connection to grant DApp access | Requires user connection approval before granting DApp access to specific RPC methods. |
| [WSB-PERM-009](#wsb-perm-009) | User confirmation before switching chains | Requires user confirmation before accepting requests to switch the active chain. |
| [WSB-PERM-010~Mobile](#wsb-perm-010mobile) | User confirmation for embedded browser requests | Requires user confirmation before processing specific RPC methods from DApps in the embedded browser. |
| [WSB-PERM-010~Browser](#wsb-perm-010browser) | User confirmation before processing requests | Requires user confirmation before processing DApp requests for specific RPC methods. |

---

### WSB-PERM-001

**SIWE domain mismatch warning**

Alerts users of discrepancies or unusual patterns in Sign-in with Ethereum (EIP-4361) requests.

**Testing instructions:** The wallet warns users of a domain or scheme mismatch when signing an EIP-4361 (Sign-In with Ethereum, SIWE) message. To verify this, generate and attempt to sign a SIWE message from a DApp whose origin differs from the domain specified in the SIWE object. The wallet passes the test if it warns the user or blocks the signing of the message.

**Reasoning:** A mismatch between the DApp origin and the domain inside the SIWE message can indicate phishing or session hijacking attempt. Warning on this helps users avoid signing login messages that could authenticate them to a different service than the one they think they are using.

---

### WSB-PERM-002~Mobile

**Wallet unlock before requests** · Mobile

Requires users to unlock before processing DApp requests when in a locked state.

**Testing instructions:** If the wallet implements WalletConnect, it requires users to unlock it to process requests from DApps to the following RPC endpoints when in a locked state: `wallet_addEthereumChain`, `wallet_watchAsset`, `eth_decrypt`, `eth_getEncryptionPublicKey`, `eth_signTypedData*`, `personal_sign`, `eth_sendTransaction`. To test this, with the wallet previously connected but in a locked state, send these RPC requests from a DApp. A positive result occurs when, for every request sent, the wallet prompts the user to provide their password to unlock it before fulfilling the request. Any bypass or leak indicates a failed test.

**Reasoning:** A locked wallet must never continue serving sensitive requests, since that would weaken the purpose of the lock state. Requiring an unlock prevents unauthorized access to accounts, signatures, or encrypted material if the device is unattended or temporarily exposed.

---

### WSB-PERM-002~Browser

**Wallet unlock before requests** · Browser

Requires users to unlock before processing DApp requests when in a locked state.

**Testing instructions:** When in a locked state, the wallet must require the user to unlock it before processing any DApp request to the following RPC endpoints: `wallet_addEthereumChain`, `wallet_watchAsset`, `eth_decrypt`, `eth_getEncryptionPublicKey`, `eth_signTypedData*`, `personal_sign`, and `eth_sendTransaction`. To test this, ensure the wallet is already connected to a DApp but remains locked, then send requests from the DApp to each of these RPC endpoints. The test passes only if every request triggers a prompt requiring the user to enter their password and unlock the wallet before the request is processed. The test fails if any request is processed while the wallet remains locked, or if any response, sensitive data, cryptographic material, or other information related to the request or wallet state is returned or leaked before the wallet is unlocked.

**Reasoning:** A locked wallet must never continue serving sensitive requests, since that would weaken the purpose of the lock state. Requiring an unlock prevents unauthorized access to accounts, signatures, or encrypted material if the device is unattended or temporarily exposed.

---

### WSB-PERM-003

**Different ChainID**

Alerts users or rejects requests to sign typed structured data (EIP-712) with a chain ID different from the active chain.

**Testing instructions:** The wallet refuses or at least warns the user when attempting to sign an EIP-712 object with a `chainId` that does not match the currently active chain. This can be verified by crafting an EIP-712 object with a `chainId` different from the wallet's current network. For example, if the wallet is currently connected to Polygon (`chainId` 137), it fails the check if it allows signing an EIP-712 object that specifies a `chainId` != 137.

**Reasoning:** EIP-712 signatures are domain-separated partly through the `chainId`, so signing data for a different chain can enable cross-chain confusion or replay-like abuse in poorly designed integrations. Warning or rejecting mismatched `chainId`s helps users understand what environment they are authorizing and prevents signatures from being produced under misleading assumptions.

---

### WSB-PERM-004

**Connected DApps**

Allows users to list and revoke connected DApps.

**Testing instructions:** The wallet lists connected DApps and allows effective access revocation. Look for a functionality within the wallet UI that lists all the connected DApps. Disconnect the testing DApp and attempt executing multiple RPC requests, to make sure no misconfiguration is left over. The lack of this functionality to both list and disconnect DApps, or any misconfiguration left over indicates a failed test.

**Reasoning:** Persistent DApp connections represent an ongoing trust relationship, so users need visibility into which apps still have access. The ability to review and revoke those connections reduces the risk of stale approvals, forgotten sessions, or continued access after the user no longer trusts the DApp.

---

### WSB-PERM-005

**List and revoke token approvals**

Allows users to view and revoke token approvals.

**Testing instructions:** The wallet offers the ability to list and revoke token approvals, via in-app functionality or links to external DApps.

**Reasoning:** Token approvals can outlive a single transaction and are a common source of loss when a spender contract is compromised or malicious. Letting users inspect and revoke approvals gives them a practical way to reduce standing risk and clean up excessive allowances.

---

### WSB-PERM-006

**Disables `eth_sign` method**

Restricts the use of the deprecated and insecure `eth_sign` method by default.

**Testing instructions:** The wallet does not support the `eth_sign` method, or it is disabled by default. With the wallet connected and unlocked, send an `eth_sign` RPC request via a DApp. If the wallet prompts a request to sign a given payload and the signature matches an `eth_sign` result, the wallet did not pass the test. Note that some wallets silently replace or redirect `eth_sign` calls to `personal_sign` instead.

**Reasoning:** `eth_sign` is historically dangerous because it signs raw data in a way that is easy to misuse and hard for users to interpret safely. In particular, it can be abused to trick users into signing raw transaction hashes or other opaque payloads without clear context. Disabling it by default reduces compatibility with insecure legacy flows and pushes DApps toward safer, more explicit signing methods.

---

### WSB-PERM-007~Mobile

**Confirmation for WalletConnect requests** · Mobile

Requires user confirmation for WalletConnect requests before granting DApp access to specific RPC methods.

**Testing instructions:** If the wallet supports WalletConnect, it requires user confirmation before processing each request to the following RPC endpoints: `wallet_addEthereumChain`, `wallet_watchAsset`, `eth_decrypt`, `eth_getEncryptionPublicKey`, `eth_signTypedData*`, `personal_sign`, `eth_sendTransaction`. If the wallet doesn't support WalletConnect, or it requires user confirmation for each RPC endpoint, the check is considered as passed.

**Reasoning:** Requiring confirmation for sensitive RPC methods ensures the DApp cannot silently trigger signing, encryption, asset watching, or transaction actions without the user noticing.

---

### WSB-PERM-008~Browser

**Connection to grant DApp access** · Browser

Requires user connection approval before granting DApp access to specific RPC methods.

**Testing instructions:** The wallet requires user approval (connection) to grant DApp access to the following RPC endpoints: `eth_accounts`, `eth_decrypt`, `eth_getEncryptionPublicKey`, `eth_signTypedData*`, `personal_sign`, `eth_sendTransaction`. For each request to these endpoints, the wallet should either prompt a DApp connection request or directly reject the request. Any bypass or data leak means the wallet did not pass the test.

**Reasoning:** A DApp should not be able to access accounts or request sensitive operations before the user explicitly grants it permission. Enforcing a connection step prevents passive data exposure and makes the trust decision visible and deliberate.

---

### WSB-PERM-009

**User confirmation before switching chains**

Requires user confirmation before accepting requests to switch the active chain.

**Testing instructions:** The wallet must require user confirmation before processing the `wallet_switchEthereumChain` RPC request from the DApp. To verify this, send a `wallet_switchEthereumChain` RPC request from a DApp. A successful test occurs when the wallet prompts the user for confirmation before switching chains. If the wallet does not prompt for confirmation, verify whether it allows users to edit or manage pre-approved networks within the connection dialog. Before performing this verification, ensure that only one network is selected or enabled in the editable list, and then attempt to switch to **any** other chain. If the wallet provides a way to modify pre-approved networks **and it either prompts for confirmation or rejects switching to chains that are not pre-approved**, the behavior is considered acceptable and the test passes. If the wallet does not offer a way to modify pre-approved networks, the test fails. This test should be performed through both the browser interface and the WalletConnect interface, if applicable.

**Reasoning:** Chain switching changes the execution context of future transactions and signatures, so it should never happen silently. Confirmation protects users from being moved onto an unexpected network where balances, assets, or contract interactions may behave very differently.

---

### WSB-PERM-010~Mobile

**User confirmation for embedded browser requests** · Mobile

Requires user confirmation before processing specific RPC methods from DApps loaded by the embedded browser.

**Testing instructions:** If the wallet has an embedded browser, it requires user confirmation before processing each DApp request to the following RPC endpoints: `wallet_addEthereumChain`, `wallet_watchAsset`, `eth_decrypt`, `eth_getEncryptionPublicKey`, `eth_signTypedData*`, `personal_sign`, `eth_sendTransaction`.

**Reasoning:** Requiring confirmation for sensitive RPC methods ensures the DApp cannot silently trigger signing, encryption, asset watching, or transaction actions without the user noticing.

---

### WSB-PERM-010~Browser

**User confirmation before processing requests** · Browser

Requires user confirmation before processing requests from DApps for specific RPC methods.

**Testing instructions:** If the wallet has an embedded browser, it requires user confirmation before processing each DApp request to the following RPC endpoints: `wallet_addEthereumChain`, `wallet_watchAsset`, `eth_decrypt`, `eth_getEncryptionPublicKey`, `eth_signTypedData*`, `personal_sign`, `eth_sendTransaction`.

**Reasoning:** Sensitive RPC methods can lead to asset movement, approval of malicious contracts, disclosure of encryption material, or misleading signatures. Requiring per-request confirmation ensures the user reviews each action in context instead of the DApp operating with broad invisible permission.

---

## Intent Verification (VERI)

Measures the wallet's ability to provide clear, human-readable transaction summaries so you know what will happen with your assets before approving.

| ID | Name | Description |
|---|---|---|
| [WSB-VERI-001](#wsb-veri-001) | Invalid checksums | Warns users when they input addresses with invalid EIP-55 checksums. |
| [WSB-VERI-002](#wsb-veri-002) | Clickable links | Consistently provides clickable links to reputable explorers for all key blockchain identifiers. |
| [WSB-VERI-003](#wsb-veri-003) | Transaction simulation | Allows users to preview the exact outcome of a transaction before signing. |
| [WSB-VERI-004](#wsb-veri-004) | Parses EIP-712 objects | Clearly displays human-readable details for EIP-712 signature requests from well-known DApps. |
| [WSB-VERI-005](#wsb-veri-005) | Clear interface for ERC-20 Approve | Clearly displays all key details for ERC-20 Approve requests. |
| [WSB-VERI-006](#wsb-veri-006) | Clear method data to be signed | Clearly displays all signature request details without truncating or hiding information. |
| [WSB-VERI-007](#wsb-veri-007) | Review data before signing | Requires users to scroll through all signature request details before being allowed to proceed. |

---

### WSB-VERI-001

**Invalid checksums**

Warns users when they input addresses with invalid EIP-55 checksums.

**Testing instructions:** The wallet, at minimum, warns users when providing addresses with invalid checksums (EIP-55 address checksum). This can be confirmed by attempting to send a transaction to an address with an incorrect checksum. To acquire an address with a faulty checksum, simply take any checksummed address and alter the case of any letter (e.g. `0x873050043AF661fe9d5633369B10139eb7b4Da54` → `0x873050043AF661fe9d5633369B10139eb7b4da54`).

**Reasoning:** EIP-55 checksums are meant to catch address-entry mistakes that would otherwise be hard to spot. Warning on an invalid checksum reduces the chance of sending funds to a mistyped or malformed address, especially when users copy addresses from unreliable sources.

---

### WSB-VERI-002

**Clickable links**

Consistently provides clickable links to reputable explorers for all key blockchain identifiers.

**Testing instructions:** The wallet provides clickable links to addresses, contracts and transaction hashes when attempting to send a transaction or querying the wallet history.

**Reasoning:** Explorer links let users independently verify addresses, contracts, and transactions instead of trusting only the wallet UI. This improves transparency and makes it easier to inspect counterparties, token contracts, and transaction outcomes before or after signing.

---

### WSB-VERI-003

**Transaction simulation**

Allows users to preview the exact outcome of the requested signature by simulating the transaction in advance.

**Testing instructions:** The wallet leverages transaction simulation. When attempting to sign a transaction and before executing it, the wallet should clearly inform the expected inputs and outputs from the execution of such transaction. An easy way of testing whether a wallet offers transaction simulation is by attempting to provide liquidity into a liquidity pool (e.g. Uniswap).

**Reasoning:** Many dangerous transactions only become obvious when looking at their actual effects, not just calldata or method names. Simulation helps users see expected token movements, approvals, or state changes in advance, which is one of the strongest defenses against deceptive DApp flows.

---

### WSB-VERI-004

**Parses EIP-712 objects**

Clearly displays human-readable details for typed structured data (EIP-712) signature requests from well-known DApps.

**Testing instructions:** The wallet parses EIP-712 objects for well-known contracts and protocols, such as detailing OpenSea Seaport listings, ERC-20 Permits, or Uniswap positions instead of plain JSON displays. To pass this test, the wallet should help users interpret what they are about to sign rather than just showing plain information.

**Reasoning:** EIP-712 is designed to make structured data more understandable, but that only works if the wallet parses and presents it clearly. Human-readable rendering helps users distinguish between harmless messages and signatures that authorize critical permissions or off-chain actions.

---

### WSB-VERI-005

**Clear interface for ERC-20 Approve**

Clearly displays all the key details for ERC-20 Approve requests.

**Testing instructions:** The wallet provides a clear interface for users to review an Approve transaction's details. It clearly informs to whom the tokens are being approved, the amount of tokens, and for which token (symbol and/or contract address for well-known tokens like USDC, or contract address for custom ERC-20s). A wallet that only shows the function parameters in hex and does not clearly inform the token being approved, the spender, or the amount should be considered a failed test. This check should be tested in two ways: 1) deploy a custom ERC-20 and call `approve`, and 2) access any swap DApp, connect the wallet and trigger an ERC-20 `approve`.

**Reasoning:** ERC-20 approvals can grant a spender long-lived control over tokens. Clearly showing the token, spender, and amount helps users detect malicious approvals, unlimited allowances, and approvals targeting unexpected contracts.

---

### WSB-VERI-006

**Clear method data to be signed**

Clearly displays all signature request details without truncating or hiding information.

**Testing instructions:** The wallet provides a clear method for users to check the data to be signed. Ensure the wallet displays all the information sent in `personal_sign` or any `eth_signTypedData` requests, without truncating any information or hiding any details, allowing the user to scrutinize and scroll through the information to be signed. For EIP-712 objects, make sure the wallet also displays the `EIP712Domain` information in full, or at least shows or interprets the `verifyingContract` address. The tester should attempt providing large inputs to these RPC endpoints.

**Reasoning:** Users cannot give meaningful consent if important parts of the message are hidden, truncated, or impossible to inspect. Full visibility into signed data is critical because attackers can bury harmful payloads or structured objects.

---

### WSB-VERI-007

**Review data before signing**

Requires users to scroll through all signature request details before being allowed to proceed with signing.

**Testing instructions:** The wallet requires users to review all data before enabling the sign button. When sending a large payload via `personal_sign` or `eth_signTypedData`, confirm that the wallet does not allow signing without first scrolling through the full content.

**Reasoning:** Forcing the user to scroll through the full request makes rushed or accidental approvals less likely. It is not a perfect defense, but it raises friction in exactly the places where attackers benefit from users clicking through without reviewing the content.

---

## Threat Prevention (THRE)

Checks that the wallet is integrated with up-to-date lists of known threats and conducts real-time checks of blockchain addresses and web domains before any transactions or connections.

| ID | Name | Description |
|---|---|---|
| [WSB-THRE-001](#wsb-thre-001) | Verified URLs | Informs users when interacting with a well-known DApp URL. |
| [WSB-THRE-002](#wsb-thre-002) | Alerts on phishing address | Prevents or alerts users about interactions with known malicious blockchain addresses. |
| [WSB-THRE-003](#wsb-thre-003) | Alerts on phishing DApps | Alerts users when attempting to interact with a known malicious URL. |
| [WSB-THRE-004](#wsb-thre-004) | DApp connection information | Informs users that connecting grants DApps access to balances, history, and signatures. |
| [WSB-THRE-005](#wsb-thre-005) | Interacting with unknown address | Warns users when interacting with unknown addresses. |
| [WSB-THRE-006](#wsb-thre-006) | Shows DApp origin URL | Clearly displays the full DApp URL in the connection prompt. |
| [WSB-THRE-007](#wsb-thre-007) | Hides spam or scam tokens and NFTs | Hides malicious tokens and NFTs by default. |

---

### WSB-THRE-001

**Verified URLs**

Informs users when interacting with a well-known DApp URL.

**Testing instructions:** The wallet informs users when they are interacting with well-known, verified URLs (DApps). In the DApp connection request screen, the wallet should indicate whether the DApp is verified or not. For instance, if attempting to connect to Uniswap, it should display a distinguishing indicator to give the user more confidence. Note that the verification indicator should only be displayed for well-known DApps, not for every DApp.

**Reasoning:** Verified DApp indicators help users distinguish trusted, well-known services from lookalike phishing sites. This is especially valuable during connection prompts, where users are often making a quick trust decision with limited information.

---

### WSB-THRE-002

**Alerts on phishing address**

Prevents or alerts users about interactions with known malicious blockchain addresses.

**Testing instructions:** The wallet prevents or alerts about interactions with known phishing or scam addresses. Such protection can be tested by sending a transaction to a well-known address associated with malicious or criminal activity. The test should include sending a transaction through both a DApp and the wallet UI. If the wallet flags any address as malicious, it should be considered a pass.

**Reasoning:** Many scams rely on users sending funds to addresses already known to be linked to theft, laundering, or fraud. Alerting on such addresses gives users a last defensive checkpoint before irreversible on-chain transfers.

---

### WSB-THRE-003

**Alerts on phishing DApps**

Alerts users when attempting to interact with a known malicious URL.

**Testing instructions:** The wallet alerts users when attempting to connect to known phishing DApps. MetaMask and several other parties maintain a [list of phishing sites](https://github.com/MetaMask/eth-phishing-detect/blob/b68a72cc477273db3717204d5db699c11980b2f5/src/config.json). The wallet passes the test if it warns users when they visit or attempt to connect to any site included in this list. Note that Chrome also includes built-in phishing protection, which may trigger warnings for some of these sites, though not all.

**Reasoning:** Phishing websites are one of the most common ways users lose funds or sign malicious payloads. Detecting and warning on known phishing domains blocks attacks early, before the user even reaches the approval stage.

---

### WSB-THRE-004

**DApp connection information**

Informs users during the connection prompt that connecting grants DApps access to view balances, transaction history, and to request signatures.

**Testing instructions:** The wallet clearly informs users, within the connection dialog, that by connecting they are allowing the DApp to view their wallet balance and activity, as well as to request transaction approvals. To verify this, attempt connecting a DApp to the wallet and inspect the connection dialog.

**Reasoning:** Explaining the scope of access during the connection flow improves informed consent and reduces the chance that users treat connection prompts as harmless.

---

### WSB-THRE-005

**Interacting with unknown address**

Warns users when interacting with unknown addresses.

**Testing instructions:** The wallet warns users when they are not interacting with a previously known or trusted address. Previously known addresses are those with which the account has already interacted (sent a transaction or token). A trusted address could be one saved in the wallet's contact list. To pass this check, the wallet must warn or show some indicator to the user when sending a transaction to an unknown address, either through a DApp or when sending manually.

**Reasoning:** Highlighting unknown recipients creates a useful pause before sending funds or interacting with a new contract that may be malicious or mistyped.

---

### WSB-THRE-006

**Shows DApp origin URL**

Clearly displays the full DApp URL in the connection prompt.

**Testing instructions:** The wallet displays the full, non-truncated DApp origin URL in the connection dialog. To verify this, attempt to connect the wallet to a DApp and confirm that the origin URL is shown in its entirety rather than being truncated.

**Reasoning:** The origin URL is one of the most important signals for spotting phishing or impersonation attempts. Showing the full, non-truncated URL helps users notice deceptive subdomains, typosquatting, or other visual tricks that could be hidden by truncation.

---

### WSB-THRE-007

**Hides spam or scam tokens and NFTs**

Hides malicious tokens and NFTs by default.

**Testing instructions:** The wallet hides spam or scam tokens and NFTs sent to the wallet. To verify this, use a test wallet with spam NFTs and tokens. The wallet should not show all these NFTs by default. If the wallet does not list tokens and/or NFTs, it is considered to be hiding spam tokens.

**Reasoning:** Spam tokens and NFTs are commonly used for phishing, impersonation, and social engineering inside wallet portfolios. Hiding them by default reduces clutter and lowers the chance that users click malicious assets or mistake them for legitimate holdings.

---

## Physical Access (PHYS)

Evaluates the wallet's implementation of device-level security features. This includes biometric authentication (fingerprint, face ID), strong password requirements, and attempt limitations.

| ID | Name | Description |
|---|---|---|
| [WSB-PHYS-001](#wsb-phys-001) | Copy mnemonics to clipboard | Minimizes exposure of secrets when copying seed phrases to the clipboard or taking screenshots. |
| [WSB-PHYS-002](#wsb-phys-002) | Robust authentication | Employs the strongest available authentication mechanisms, including biometrics and strong passwords. |
| [WSB-PHYS-003](#wsb-phys-003) | Warns about sharing mnemonics/private keys | Warns users of the risk before allowing access to seed phrases or private keys. |
| [WSB-PHYS-004](#wsb-phys-004) | Manual wallet lock | Allows users to lock the wallet manually. |
| [WSB-PHYS-005](#wsb-phys-005) | Seed phrase authentication | Requires authentication to access seed phrases or private keys. |
| [WSB-PHYS-006~Mobile](#wsb-phys-006mobile) | Automatic wallet lock | Automatically locks after a period of inactivity. |
| [WSB-PHYS-006~Browser](#wsb-phys-006browser) | Automatic wallet lock | Automatically locks after a period of inactivity. |

---

### WSB-PHYS-001

**Copy mnemonics to clipboard**

Minimizes exposure of secrets by limiting or warning users when copying seed phrases to the clipboard or taking screenshots.

**Testing instructions:** When copying mnemonics to the clipboard, the wallet limits the time these secrets are present in the clipboard (1 minute max), warns users about the risks of copying to clipboard, or prevents users from copying to clipboard. On mobile, it also warns users about the risks of taking screenshots or prevents users from taking screenshots.

**Reasoning:** Seed phrases are extremely sensitive, and clipboards and screenshots are common exfiltration paths on both desktop and mobile systems. Limiting, warning, or blocking these actions reduces the exposure window and makes users think twice before handling secrets insecurely.

---

### WSB-PHYS-002

**Robust authentication**

Employs the strongest available authentication mechanisms, including biometrics, login attempt rate-limiting, and enforcement of strong passwords.

**Testing instructions:** The wallet utilizes the most robust authentication mechanism supported by the platform. On mobile platforms, wallets should support biometrics; if they allow easy-to-guess passwords or PINs, make sure that attempts are rate-limited after the 5th failure. On browser extension wallets, require at least an 8-character password and verify that it does not accept passwords such as `12345678`, `qwertyui`, or `password`.

**Reasoning:** Weak local authentication can completely undermine all other security controls. Strong passwords, biometrics, and rate-limiting make brute force, shoulder surfing, and opportunistic device access much harder.

---

### WSB-PHYS-003

**Warns about sharing mnemonics/private keys**

Warns users of the risk before allowing access to seed phrases or private keys.

**Testing instructions:** The wallet warns users about the risks associated with sharing or revealing mnemonics/private keys, to ensure they understand the implications of their actions. When attempting to reveal these secrets, the wallet should clearly communicate these risks and display a warning to the user to pass the test.

**Reasoning:** Users sometimes reveal seed phrases or private keys without fully understanding that doing so grants complete control over their assets. A clear warning at the moment of exposure can prevent catastrophic mistakes and reinforces the severity of the action.

---

### WSB-PHYS-004

**Manual wallet lock**

Allows users to lock the wallet manually.

**Testing instructions:** The wallet features a lock button to lock it manually. Lack of a lock button means a failure to pass this check.

**Reasoning:** Users need a fast way to secure the wallet when stepping away, sharing a screen, or using a device in public. Manual locking gives immediate control rather than forcing the user to wait for an inactivity timer.

---

### WSB-PHYS-005

**Seed phrase authentication**

Requires authentication to access seed phrases or private keys.

**Testing instructions:** If the wallet supports seed-phrase backup functionality, it enforces authentication by default to display the mnemonics or private keys. Showing these secrets without requiring any authentication by default translates into a failed test.

**Reasoning:** Seed phrases and private keys should be protected behind strong re-authentication because they are the ultimate account recovery and control mechanism. Displaying them without an authentication step makes accidental exposure and local compromise far more likely.

---

### WSB-PHYS-006~Mobile

**Automatic wallet lock** · Mobile

Automatically locks after a period of inactivity.

**Testing instructions:** The wallet auto-locks on inactivity (1 minute or less by default), when the device gets locked, or when the app is moved to background execution. Check the wallet settings to set the auto-lock time, which should not exceed 1 minute. If it's set to 0, the wallet won't auto-lock. Some wallets have a fixed auto-lock period not shown in settings; to check this, exit the wallet and see if it locks once a reasonable time has elapsed.

**Reasoning:** Mobile devices are frequently lost, borrowed, or left unlocked briefly, so short auto-lock periods materially reduce the chance of unauthorized access.

---

### WSB-PHYS-006~Browser

**Automatic wallet lock** · Browser

Automatically locks after a period of inactivity.

**Testing instructions:** The wallet auto-locks on inactivity (20 minutes or less). To test this feature, go through the wallet settings and see if the auto-lock time can be configured. If it's 0 by default, then the wallet does not auto-lock on inactivity. Some wallets have a pre-determined fixed 20-minute auto-lock period that does not appear in the settings, and therefore the only way of verifying this is letting the wallet sit inactive for 20 minutes.

**Reasoning:** Browser extension wallets often remain open for long periods on shared or unattended computers. A reasonable auto-lock time reduces the window in which an attacker, coworker, or malicious webpage can exploit an already-unlocked wallet.
