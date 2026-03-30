# Wallet Security Framework (WSF)

This repository contains two complementary resources for evaluating the security of software web3 wallets, developed by [Coinspect](https://www.coinspect.com).

Important Note: This framework for crypto wallets security is product of ongoing research and is inherently dynamic in nature. While every effort has been made to ensure its accuracy and applicability, it should not be viewed as definitive. It's recommended to use this guide in conjunction with other established security standards to provide a more comprehensive security posture.

## [Wallet Security Controls (WSC)](WSC/README.md)

A catalog of security controls aimed at developers and auditors reviewing wallet implementations. It covers areas such as key management, authentication, provider injection, and secure coding practices. Most controls are white-box in nature and require access to the wallet's source code or internal configuration, though some can be verified dynamically.

## [Wallet Security Benchmark (WSB)](WSB/README.md)

The WSB is a set of security tests that can be performed against a web3 wallet without access to its source code (black-box). Currently, the tests cover two main areas:
- phishing protection, evaluating how well a wallet defends users against deceptive dApps and malicious signature requests;
- physical security, assessing wallet behavior when an attacker has physical access to the user's device.

Because the tests are interaction-based — observing how a wallet responds to real interactions — they can be performed by anyone, not just security experts or developers. This makes the methodology applicable across the most widely used crypto wallets, repeatable, scalable, and eventually automatable.