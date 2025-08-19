# Wallet Security Verification Standard (WSVS)

This is a reference list of security checks for cryptocurrency wallets, aiming to assist technical professionals in enhancing wallet security against common threats, including phishing attacks.

The checks below are part of ongoing research conducted by [Coinspect](https://www.coinspect.com) to identify prevalent flaws and specify best practices in cryptocurrency wallets. 

## Category Description

| Category           | Description                                                                                   |
|--------------------|----------------------------------------------------------------------------------------------|
| `ConsumerWallet`    | End-user wallets (browser/mobile/desktop) for personal use                                   |
| `HotWallet`         | Wallets connected to the internet, typically for exchanges or operational use                |
| `PaymentGateway`    | Wallets used for automated payment processing and merchant services                          |
| `EnterpriseCustody` | Institutional custody solutions, often with advanced controls (HSM, MPC, etc.)              |

## Security checks

Most of the checks presented in the tables below apply broadly and aren't limited to a specific platform. However, some checks are relevant only for browser extensions or applications that inject a provider. In addition, a few checks aim to EVM-compatible wallets only.

Checks are classified by categories (General, Authentication, etc.), and provide additional information in hyperlinks.

Important Note: This standard for crypto wallets security is product of ongoing research and is inherently dynamic in nature. While every effort has been made to ensure its accuracy and applicability, it should not be viewed as definitive. It's recommended to use this guide in conjunction with other established security standards, such as the [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/), to provide a more comprehensive security posture.

### 1. General

|  #  | Description | Comment | Attacks/Reports | Category        |
| --- | ----------- | ------- | --------------- | --------------  |
| 1.1 | Ensure the extension only requires permissions essential for its operation. See [LINK1](https://developer.chrome.com/docs/extensions/mv2/declare_permissions/) for V2 permissions. | Extension oriented | | `ConsumerWallet` |
| 1.2 | Verify `web_accessible_resources` entry on `manifest.json` doesn’t contain HTML resources that can interact with the wallet or redirect to `chrome-extension://`. See [LINK1](https://developer.chrome.com/docs/extensions/mv3/manifest/web_accessible_resources/) and [LINK2](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/manifest.json/web_accessible_resources#security). | Extension oriented | | `ConsumerWallet` |
| 1.3 | Verify it doesn't leak any private information to known trackers/metrics packages or any custom API that could identify the user, such as mnemonics, user passwords, or addresses. | | [Swisstronik Wallet](https://github.com/SigmaGmbH/Bug-Bounty-1.0/issues/11) [Slope Finance](https://slope-finance.medium.com/slope-wallet-sentry-vulnerability-digital-forensics-and-incident-response-report-d7a5904e5a39) | `ConsumerWallet` |
| 1.4 | Verify that the content script is injected before any malicious extension modifies the DOM. Ensure the `run_at` attribute is set to `document_start` in the content script. | Extension oriented | | `ConsumerWallet` |

### 2. Authorization

|  #  | Description | Comment | Attacks/Reports | Category        |
| --- | ----------- | ------- | --------------- | --------------  |
| 2.1 | Verify that DApps cannot access sensitive endpoints without first the user approving the connection to the wallet. For clarity, `eth_chainId` should be safe, but `eth_accounts` poses a privacy risk. | | | `ConsumerWallet` |
| 2.2 | Verify wallet requires users to approve transaction or message signing requests originated from the DApp. | | | `ConsumerWallet` |
| 2.3 | When the wallet is locked and a DApp sends a request, the wallet should first prompt the user to unlock it. For instance, if the wallet is locked, the DApp directly invokes a dialogue for the user to authorize a transaction. | | | `ConsumerWallet` |
| 2.4 | Ensure the wallet doesn't allow methods that sign arbitrary data, like Metamask's older `eth_sign` function. If such a method exists, the user should receive a warning before proceeding. | | | `ConsumerWallet` |
| 2.5 | Verify wallet requires user consent before loading remote URLs (eg. NFTs metadata) to prevent revealing association between IP and wallet addresses. | | | `ConsumerWallet` |
| 2.6 | Verify the wallet provides mechanisms to filter spam NFTs. | | | `ConsumerWallet` |

### 3. Authentication

|  #  | Description | Comment | Attacks/Reports | Category        |
| --- | ----------- | ------- | --------------- | --------------  |
| 3.1 | Verify that users can manually lock the wallet. | | | `ConsumerWallet` |
| 3.2 | Ensure that the wallet utilizes the most robust authentication mechanism supported by the device. When available, prefer using the device biometrics API instead of text passwords. | | | `ConsumerWallet` |
| 3.3 | Verify the wallet enforces a password policy that emphasizes passphrase length rather than complexity. | | | `ConsumerWallet` |
| 3.4 | Verify wallet auto-locks on inactivity. Consider default lock time not higher than 15 min. | | | `ConsumerWallet` |

### 4. User Interface

|  #  | Description | Comment | Attacks/Reports | Category        |
| --- | ----------- | ------- | --------------- | --------------  |
| 4.1 | Verify that the user interface displays a warning in the 'Connect' dialog when a user interacts with a DApp for the first time. | | | `ConsumerWallet` |
| 4.2 | Verify that the wallet clearly informs users, within the connection dialog, that by connecting they are allowing the DApp to view their wallet balance and activity, as well as to request transaction approvals. | | | `ConsumerWallet` |
| 4.3 | Verify the wallet informs users when they are attempting to connect to an unknown DApp. Ideally, the wallet should also inform users when they are interacting with official DApps. | | | `ConsumerWallet` |
| 4.4 | Verify the wallet prevents users from connecting to known phishing DApps. | | | `ConsumerWallet` |
| 4.5 | Verify the wallet informs users when they are attempting to interact with an unknown address. Ideally, the wallet should also inform users when they are interacting with well known addresses. | | | `ConsumerWallet` |
| 4.6 | Verify the wallet prevent users from interacting with known phishing or scam addresses. | | | `ConsumerWallet` |
| 4.7 | Verify the UI warns the user when interacting with an address for the first time. | | | `ConsumerWallet` |
| 4.8 | Verify the UI displays a list of connected DApps and allows the user to revoke access. | | | `ConsumerWallet` |
| 4.9 | Verify the UI displays the ENS address associated to destination address. | | | `ConsumerWallet` |
| 4.10 | Verify the UI transaction dialogs offer a user-friendly way to confirm transaction details, at least: contract address, function, spender, amount, gas price/limit. Include descriptive messages depending on contract calls, for example in the case of an approval, the message could indicate that the user is granting permission to a contract to spend a given amount of tokens on their behalf. | | | `ConsumerWallet` |
| 4.11 | Ensure that the UI dialogs offer a straightforward method for users to confirm the data they are signing. For instance, when dealing with [EIP-712](https://eips.ethereum.org/EIPS/eip-712) signatures, the interface should display the `EIP712Domain` information and exclusively the struct intended for signing. | | | `ConsumerWallet` |
| 4.12 | Wallet dialogs must clearly differentiate between DApp-sourced information and native wallet details to guard against phishing threats. For example, without this distinction, attackers could misuse overly long DApp titles to flood the transaction dialog with deceptive content. | | | `ConsumerWallet` |
| 4.13 | Verify the UI parses [EIP-712](https://eips.ethereum.org/EIPS/eip-712) objects for well-known contracts or protocols. For example, parsing and explaining Opensea Seaport listings in detail rather than showing a plain JSON message. | | | `ConsumerWallet` |
| 4.14 | Verify the UI does not truncate the DApp domain URL (e.g. `mysite.com.attacker.com` is displayed as `mysite.com`) | | | `ConsumerWallet` |
| 4.15 | The wallet must require users to carefully review all data before enabling the sign button. | | | `ConsumerWallet` |
| 4.16 | Verify the UI includes clickable links to the explorer for the displayed addresses and transactions (e.g. when interacting with a contract, when viewing transactions history, etc). | | | `ConsumerWallet` |
| 4.17 | Verify the UI warns users before copying mnemonics to the clipboard and encourage them to backup it outside the computer in a secure place. | | | `ConsumerWallet` |
| 4.18 | The mobile wallet should prevent users from taking screenshots of the mnemonic phrase. This prevents it from being inadvertently stored in a cloud backup, for instance. | | | `ConsumerWallet` |
| 4.19 | Verify the UI obfuscates mnemonic phrases by default at any time. | | | `ConsumerWallet` |
| 4.20 | Verify the wallet allows displaying truncated data in full (eg. truncated addresses or rounded values). | | | `ConsumerWallet` |
| 4.21 | Verify the wallet alerts users when trying to connect to a DApp whose URL includes deceptive, lookalike characters (homograph words or cyrillic characters). | | | `ConsumerWallet` |

### 5. Ethereum Improvement Proposal (EIP) Implementation

|  #  | Description | Comment | Attacks/Reports | Category        |
| --- | ----------- | ------- | --------------- | --------------  |
| 5.1 | Verify wallet implements cross-chain replay protection ([EIP-1559](https://eips.ethereum.org/EIPS/eip-1559), transaction type 2). | | | `ConsumerWallet` |
| 5.2 | From [EIP-712](https://eips.ethereum.org/EIPS/eip-712), verify "The user-agent should refuse signing if chainId does not match the currently active chain". Otherwise, the wallet should at least warn the user when DApp is trying to use a chainId different from the active chain. | | | `ConsumerWallet` |
| 5.3 | Verify that it warns users when providing addresses with invalid [EIP-55](https://eips.ethereum.org/EIPS/eip-55) checksums. | | | `ConsumerWallet` |
| 5.4 | Verify that it warns users when providing addresses without [EIP-55](https://eips.ethereum.org/EIPS/eip-55) checksum. | | | `ConsumerWallet` |

### 6. Communication

|  #  | Description | Comment | Attacks/Reports | Category        |
| --- | ----------- | ------- | --------------- | --------------  |
| 6.1 | Verify that the extension injects into [Secure Contexts](https://developer.mozilla.org/en-US/docs/Web/Security/Secure_Contexts) only. | Extension oriented | | `ConsumerWallet` |
| 6.2 | Verify that the message listener in the content script validates the origin of incoming messages to make sure they originate from the DApp. At load time, check that messages match the DApp domain and window (`event.source == window` and `event.origin == window.location.origin`). | Extension oriented | | `ConsumerWallet` |
| 6.3 | Verify that messages received by the content script message listener are considered untrusted and validated before passing it to background script or service worker (not passed directly to `port.postMessage`, `runtime.sendMessage` or `tabs.sendMessage`). | Extension oriented | | `ConsumerWallet` |
| 6.4 | Ensure that input sanitization and authorization aren't executed in the MAIN runtime environment (i.e. the provider), as a malicious DApp can bypass these measures by sending a message/event directly to the Content Script. | Extension oriented | | `ConsumerWallet` |
| 6.5 | Verify that communication with other extensions or websites is not permitted (avoid using `runtime.onMessageExternal` and `runtime.onConnectExternal`). | Extension oriented | | `ConsumerWallet` |
| 6.6 | Verify wallet validates the schema, host and port of the connected dapp's URL. Check it doesn't allow by default DApps running on different ports or subdomains. (e.g. `subdomain.dapp.com` is not accepted as `dapp.com`). | Extension oriented | | `ConsumerWallet` |

### 7. Storage

|  #  | Check | Comment | Attacks/Reports | Category        |
| --- | ----- | ------- | --------------- | --------------  |
| 7.1 | Ensure that the seed is encrypted using a robust algorithm and a secure passphrase. | | | `ConsumerWallet` |
| 7.2 | Ensure the wallet uses reputable and secure cryptographic libraries, avoiding custom cryptographic code. | | | `ConsumerWallet` |
| 7.3 | Ensure the encryption key is derived from the user-provided password using a robust and computationally-intensive algorithm. | | [Swisstronik Wallet](https://github.com/SigmaGmbH/Bug-Bounty-1.0/issues/13) | `ConsumerWallet` |
| 7.4 | If enabling cloud backup, verify that stored data is encrypted using a strong passphrase. Additionaly, users should be informed about the risk of storing the private key to cloud storage. | | | `ConsumerWallet` |
| 7.5 | Key material (mnemonics/private key) must not leave the system except through a specific export flow. | | | `ConsumerWallet` |


## Contributing

The present checklist is meant to be for the community and by the community, so we invite everyone to contribute!

* If you would like to have new checks added to the checklist, please [create an issue](https://github.com/coinspect/wallet-security-checklist/issues/new) so that we can discuss the changes. Make sure to add a short generic description of the check (which would be later placed in the checklist), links, examples, and a longer description if necessary
* Also, if you feel like further improving, clarifying or correcting any of the checks published, please create an issue and we'll be happy to  discuss it
