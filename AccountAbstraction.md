A Diagrammatic History of Ethereum Account Abstraction

Note from Author
I am starting the research phase for this paper today, on December 1st 2024. I am applying for the Ethereum Foundation’s 2025 Software Engineering Internship, with a specific interest in improving Ethereum’s UX. Therefore, writing this paper will be an effective way to get an in-depth understanding of AA. I am already familiar with the concepts of ERC4337, including bundlers and entry-point contracts, paymasters and transaction sponsoring…

I will now go deeper, by reading EIPs 86, 101, 1613, 2771, 2938, 3074, 4337, 7701, 7702, 7720, RIP 7560 and other related sources.
Abstract
“The goal with Account Abstraction has always been to solve the cross-chain UX problem” - Yoav Weiss


Introduction


Account Abstraction (AA) involves three parts of a transaction flow:

Authorization Logic (how the transaction is approved)
Execution Logic (what the transaction does)
Fee Payment (how the miner is paid)


Following years of exploration with the EIPs described in this paper, EIP-4337 condenses the key goal of Account Abstraction into: allowing users to not have EOAs, and instead “use smart contract wallets containing arbitrary verification logic”. This is a technical, practical, operational view of the goal.

On the other hand, the normative end goal has always been to improve UX. Specifically, the three improvements listed in the Pros and Cons tables in this paper are:
Signing UX
Wallet Recovery
Gas Payments
This can be further abstracted into the real final goals:
Improving onboarding and usage friction by reducing the number of steps and concepts users need to understand when using dApps.
Giving users peace of mind regarding fears of having their seed phrase lost or stolen.






Traditional Transaction Flow

Before delving into the many EIPs that have emerged to address the AA goals, we must understand what transactions are and the traditional flow of transactions on Ethereum.

A transaction is a write operation - i.e. an action that alters the onchain state. There are two types of transactions on Ethereum: Ether transfers (from EOA to EOA) or function calls (from EOA to Contract).

The process is kicked off by a user operating an Externally Owned Account (EOA), which is a self-custodial wallet. They set all information about their desired transaction and share this with an RPC Node.
A Remote Procedure Call (RPC) Node allows users operating EOAs (not operating their own network nodes), to execute read and write operations. 


Past Proposed AA Approaches

EIP-101 (2015)
https://eips.ethereum.org/EIPS/eip-101

EIP-101 redefines EOAs to no longer store Ether, i.e. only store code and storage. Transactions are simplified to include only the recipient, gas limit, data, and optional code for contract creation, eliminating fields like msg.value.

Ether is instead managed by a unique premined contract at address 0, which handles transfers and balances for all accounts.


Signing UX
Allows multisigs, state channels, time locks, time-based expirations…
Wallet recovery
Does not improve wallet recovery or support deterministic address creation
Gas payments
Contracts can pay gas fees directly, users don’t need Ether
Security
Custom contract logic introduces a wider attack surface
Dev overhead
Requires secure custom logic for balance management
Future proofing
Allows custom signing schemes, e.g. quantum-resistant
Validator incentives
No changes to validation fees
Implementation
Requires some protocol-layer changes










EIP-86 (2017)
https://eips.ethereum.org/EIPS/eip-86

EIP-86 introduces the idea of abstracting transaction origin (who sent it) and signature validation (how it’s authorized), or in other words, “abstracting account security”
It also introduces the CREATE2 opcode, for deterministic contract creation. Although EIP-86 was never implemented, CREATE2 was later re-proposed and finalized as EIP-1014 and introduced in the Constantinople upgrade (February 2019).


Signing UX
Allows multisigs, state channels, time locks, time-based expirations…
Wallet recovery
Allows social recovery
Gas payments
Contracts can pay gas fees directly, users don’t need Ether
Security
Custom contract logic introduces a wider attack surface
Dev overhead
Requires secure and efficient custom logic for common tasks
Future proofing
Allows custom signing schemes, e.g. quantum-resistant
Validator incentives
Makes it harder for miners to verify fee payment upfront *
Implementation
Requires some protocol-layer changes


*Ethereum was still running Proof of Work at the time of this proposal, hence the miners

EIP-1613 (2018)
https://eips.ethereum.org/EIPS/eip-1613

EIP-1613 proposes the Gas Stations Network (GSN), a decentralised offchain network of relayers, which enables users to make “collect calls”, i.e. transactions paid for by the recipient.
The relayers pay the gas fee upfront and then get reimbursed by the receiver contract (can’t be an EOA). A singleton contract RelayHub manages relayers, ensuring their authorization and reputation, and handles reimbursements.


Signing UX
Allows collect-calls to contracts
Wallet recovery
No changes to wallet recovery
Gas payments
Dapps can sponsor gas fees, users don’t need Ether (except for transfers)
Security
Reliance on relayers could be a potential point of failure
Dev overhead
Integration with the GSN adds complexity to dApp development
Future proofing
Adds some complexity
Validator incentives
No impact on validator incentives
Implementation
Requires deploying and maintaining a network of relayers



V. Buterin’s Proposed Scheme (Feb 2020)
https://ethereum-magicians.org/t/implementing-account-abstraction-as-part-of-eth1-x/4020

This proposed scheme builds on the concept of account abstraction by introducing a two-step process for transaction validation: lightweight verification and execution. This approach allows for custom account logic while maintaining network efficiency and miner incentives.

It avoids creating a new account type, relying instead on a unique ENTRY_POINT contract to route transactions and manage gas payments. It does not require major protocol changes and focuses on making advanced features accessible without sacrificing backward compatibility.

Gas payment is handled by a ney PAYGAS opcode, which deducts gas fees dynamically during transaction execution, locks payment to miners, and ensures unused gas is refunded.

Several subsequent EIPs draw inspiration from this approach, adopting the concepts of the ENTRY_POINT contract and the PAYGAS opcode.



Signing UX
Allows multisigs, state channels, time locks, time-based expirations…
Wallet recovery
Allows social recovery and deterministic wallet recreation
Gas payments
Contracts pay gas fees directly, users don’t need Ether
Security
Custom contract logic introduces a wider attack surface
Dev overhead
Requires secure and efficient custom logic for common tasks
Future proofing
Allows custom signing schemes, e.g. quantum-resistant
Validator incentives
Separates gas validation and execution, making fees predictable
Implementation
No protocol-layer changes required



ERC-2771 (Jul 2020)
https://eips.ethereum.org/EIPS/eip-2771

EIP-2771 introduces Forwarders as an onchain and offchain entity used to sponsor transactions. Users can send a free offchain meta-transaction to a dApp’s forwarder, which then sends the tx onchain to the recipient, including the sender’s address in the tx data.

This allows users to interact with decentralized applications (dApps) without holding Ether for gas fees. This approach simplifies user interactions and enhances accessibility without requiring changes to the Ethereum protocol.

N.B. ERCs are a subset of EIP. More specifically, ERCs are EIPs that define application level standards. In this case, an interface and behaviour for contracts to enable meta-transactions


Signing UX
Dapps can sponsor gas fees, users don’t need Ether (except for transfers)
Wallet recovery
No changes addressing wallet recovery
Gas payments
Contracts pay gas fees directly, users don’t need Ether
Security
Trusted forwarders could introduce centralization risks
Dev overhead
Integrating contracts with the forwarder protocol adds complexity
Future proofing
Adds some complexity
Validator incentives
No impact on validator incentives
Implementation
Requires deploying and maintaining trusted forwarder contracts



EIP-2938 (Sep 2020)
https://eips.ethereum.org/EIPS/eip-2938

EIP-2938 proposes integrating account abstraction directly into the Ethereum protocol, allowing smart contracts to function as top-level accounts capable of initiating transactions and paying fees.
Gas payment is handled by a new PAYGAS opcode, which deducts gas fees dynamically during transaction execution, locks payment to miners, and ensures unused gas is refunded.
This enables features like multisigs, social recovery, time locks, and quantum-resistant signing schemes, while maintaining miner incentives and network security.

Signing UX
Allows multisigs, state channels, time locks, time-based expirations…
Wallet recovery
Allows social recovery
Gas payments
Contracts pay gas fees directly, users don’t need Ether
Security
Custom contract logic introduces a wider attack surface
Dev overhead
Requires secure and efficient custom logic for common tasks
Future proofing
Allows custom signing schemes, e.g. quantum-resistant
Validator incentives
Makes it harder for miners to verify fee payment upfront
Implementation
Requires protocol layer changes



EIP-3074 (Oct 2020)
https://eips.ethereum.org/EIPS/eip-3074

EIP-3074 introduces two new EVM opcodes, AUTH and AUTHCALL, enabling externally owned accounts (EOAs) to delegate transaction control to smart contracts called Invokers. This allows EOAs to execute complex operations and batch transactions without deploying individual smart contracts, enhancing functionality and user experience.
Invokers can, batch multiple operations into a single transaction, sponsor gas fees, allowing users to interact without holding ETH. This reduces the technical and financial burden on users, as Invokers can handle batch transactions, gas sponsorship, conditional execution, atomic operations, cross-contract interactions…

Signing UX
Enables multisigs, batch transactions, and sponsored transactions without requiring EOAs to deploy contracts
Wallet recovery
No changes addressing wallet recovery
Gas payments
Third parties can sponsor gas, users don’t need Ether
Security
Delegating control to invokers introduces trust risks; compromised invokers can misuse authorization
Dev overhead
Reduces the need for custom contract deployment for advanced functionalities
Future proofing
Allows custom signing schemes, e.g. quantum-resistant
Validator incentives
No impact on validator incentives
Implementation
Requires protocol changes to add new opcodes



EIP-4337 (2021)
https://eips.ethereum.org/EIPS/eip-4337

EIP-4337 is the strongest approach to date, to introduce AA without many of the drawbacks. It does not require infrastructure-layer changes, instead introducing a higher-layer pseudo-transaction object called a UserOperation.
Users submit UserOperations to a separate mempool. Any block builder can be a take on the role of Bundler, which aggregates UserOperations into a single transaction that interacts with a unique EntryPoint contract,

facilitating decentralized inclusion of these operations into blocks.

allowing users to utilize smart contract wallets with custom verification logic without requiring consensus-layer protocol changes.






Short-term Future AA Approaches
Included in Pectra

EIP-3074

EIP-7702
Give EOAs some of the AA benefits: EOAs can use paymasters, i.e. pay gas with tokens or get gas sponsorship.
Long-term Future AA Approaches

EIP-7701


Conclusion



