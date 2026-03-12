# Wallet Security Standards

This repository contains two complementary resources for evaluating the security of software web3 wallets, developed by [Coinspect](https://www.coinspect.com).

## [Wallet Security Verification Standard (WSVS)](WSVS/README.md)

A checklist of security requirements aimed at developers and auditors reviewing wallet implementations. It covers areas such as key management, authentication, provider injection, and secure coding practices. Most checks are white-box in nature and require access to the wallet's source code or internal configuration, though some can be verified dynamically.

## [Wallet Security Benchmark (WSB)](WSB/README.md)

A black-box test suite for dynamic security evaluation, executable against any wallet without source code access. Tests focus primarily on phishing resistance — assessing how well a wallet protects users from deceptive DApps, malicious signature requests, and social engineering at the UI level.
