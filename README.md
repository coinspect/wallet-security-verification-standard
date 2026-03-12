# Wallet Security Verification Standard (WSVS)

This repository contains two complementary resources for evaluating the security of software web3 wallets, developed by [Coinspect](https://www.coinspect.com).

Important Note: This standard for crypto wallets security is product of ongoing research and is inherently dynamic in nature. While every effort has been made to ensure its accuracy and applicability, it should not be viewed as definitive. It's recommended to use this guide in conjunction with other established security standards, such as the [OWASP ASVS](https://owasp.org/www-project-application-security-verification-standard/), to provide a more comprehensive security posture.

## [Wallet Security Controls (WSC)](WSVS/README.md)

A catalog of security controls aimed at developers and auditors reviewing wallet implementations. It covers areas such as key management, authentication, provider injection, and secure coding practices. Most controls are white-box in nature and require access to the wallet's source code or internal configuration, though some can be verified dynamically.

## [Wallet Security Benchmark (WSB)](WSB/README.md)

A black-box test suite for dynamic security evaluation, executable against any wallet without source code access. Tests focus primarily on phishing resistance — assessing how well a wallet protects users from deceptive DApps, malicious signature requests, and social engineering at the UI level.
