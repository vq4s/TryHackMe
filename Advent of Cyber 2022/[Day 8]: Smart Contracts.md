# Day 8: Last Christmas I gave you my ETH 
## Learning Objectives

*   Explain what smart contracts are, how they relate to the blockchain, and why they are important.
*   Understand how contracts are related, what they are built upon, and standard core functions.
*   Understand and exploit a common smart contract vulnerability.

## What is a Blockchain?

One of the most recently innovated and discussed technologies is the blockchain and its impact on modern computing. While historically, it has been used as a financial technology, it's recently expanded into many other industries and applications. Informally, a blockchain acts as a database to store information in a specified format and is shared among members of a network with no one entity in control.

By definition, a blockchain is a digital database or ledger distributed among nodes of a peer-to-peer network. The blockchain is distributed among "peers" or members with no central servers, hence "decentralized." Due to its decentralized nature, each peer is expected to maintain the integrity of the blockchain. If one member of the network attempted to modify a blockchain maliciously, other members would compare it to their blockchain for integrity and determine if the whole network should express that change.

The core blockchain technology aims to be decentralized and maintain integrity; cryptography is employed to negotiate transactions and provide utility to the blockchain.

But what does this mean for security? If we ignore the core blockchain technology itself, which relies on cryptography, and instead focus on how data is transferred and negotiated, we may find the answer concerning. Throughout this task, we will continue to investigate the security of how information is communicated throughout the blockchain and observe real-world examples of practical applications of blockchain.

## Introduction to Smart Contracts

A majority of practical applications of blockchain rely on a technology known as a smart contract. Smart contracts are most commonly used as the backbone of DeFi applications (Decentralized Finance applications) to support a cryptocurrency on a blockchain. DeFi applications facilitate currency exchange between entities; a smart contract defines the details of the exchange. A smart contract is a program stored on a blockchain that runs when pre-determined conditions are met.

Smart contracts are very comparable to any other program created from a scripting language. Several languages, such as Solidity, Vyper, and Yul, have been developed to facilitate the creation of contracts. Smart contracts can even be developed using traditional programming languages such as Rust and JavaScript; at its core, smart contracts wait for conditions and execute actions, similar to traditional logic.

## Functionality of a Smart Contract

Building a smart contract may seem intimidating, but it greatly contrasts with core object-oriented programming concepts.

Before diving deeper into a contract's functionality, let's imagine a contract was a class. Depending on the fields or information stored in a class, you may want individual fields to be private, preventing access or modification unless conditions are met. A smart contract's fields or information should be private and only accessed or modified from functions defined in the contract. A contract commonly has several functions that act similarly to accessors and mutators, such as checking balance, depositing, and withdrawing currency.

Once a contract is deployed on a blockchain, another contract can then use its functions to call or execute the functions we just defined above.

If we controlled Contract A and Contract B wanted to first deposit 1 Ethereum, and then withdraw 1 Ethereum from Contract A,

Contract B calls the deposit function of Contract A.

Contract A authorizes the deposit after checking if any pre-determined conditions need to be met.

![1](https://user-images.githubusercontent.com/53142039/206532204-51514bad-de7e-430c-a146-432c798e94d4.png)

Contract B calls the withdraw function of Contract A.

Contract A authorizes the deposit if the pre-determined conditions for withdrawal are met.

![2](https://user-images.githubusercontent.com/53142039/206532213-148430a4-4412-411e-95cb-f4e9d6153de6.png)

Contract B can execute other functions after the Ether is sent from Contract A but before the function resolves.

![3](https://user-images.githubusercontent.com/53142039/206532214-6509dd79-e8a0-4730-b5a8-c0e112f695a7.png)

## How do Vulnerabilities in Smart Contracts Occur?

Most smart contract vulnerabilities arise due to logic issues or poor exception handling. Most vulnerabilities arise in functions when conditions are insecurely implemented through the previously mentioned issues.

Let's take a step back to Contract A in the previous section. The conditions of the withdraw function are,

*   Balance is greater than zero
*   Send Etherium

At first glance, this may seem fine, but when is the amount to be sent subtracted from the balance? Referring back to the contract diagram, it is only ever deducted from the balance after the Etherium is sent. Is this an issue? The function should finish before the contract can process any other functions. But if you recall, a contract can consecutively make new calls to a function while an old function is still executing. An attacker can continuously attempt to call the withdraw function before it can clear the balance; this means that the pre-defined conditions will always be met. A developer must modify the function's logic to remove the balance before another call can be made or require stricter requirements to be met.

## The Re-entrancy Attack

In the above section, we informally introduced a common vulnerability known as re-entrancy. Reiterating what was covered above, re-entrancy occurs when a malicious contract uses a fallback function to continue depleting a contract's total balance due to flawed logic after an initial withdraw function occurs.

We have broken up the attack into diagrams similar to those previously seen to explain this better.

Assuming that contract B is the attacking contract and contract A is the storage contract. Contract B will execute a function to deposit, then withdraw at least one Ether. This process is required to initiate the withdraw function's response.

![4](https://user-images.githubusercontent.com/53142039/206532221-7e4ba5ee-b1a3-4ede-ab1b-1c82a3120534.png)
Note the difference between account balance and total balance in the above diagram. The storage contract separates balances per account and keeps each account's total balance combined. We are specifically targeting the total balance we do not own to exploit the contract.

![5](https://user-images.githubusercontent.com/53142039/206532223-ddddd66b-19ca-4914-8f59-cf81a9974f99.png)

At this point, contract B can either drop the response from the withdraw function or invoke a fallback function. A fallback function is a reserved function in Solidity that can appear once in a single contract. The function is executed when currency is sent with no other context or data, for example, what is happening in this example. The fallback function calls the withdraw function again, while the original call to the function was never fully resolved. Remember, the balance is never set back to zero, and the contract thinks of Ether as its total balance when sending it, not divided into each account, so it will continue to send currency beyond the account's balance until the total balance is zero.

![6](https://user-images.githubusercontent.com/53142039/206532227-7a316656-d180-4f49-b158-77f5888bb346.png)

Because the withdraw function sends Ether with no context or data, the fallback function will be called again, and thus an infinite loop can now occur.

Now that we have covered how this vulnerability occurs and how we can exploit it, let's put it to the test! We have provided you with the contract deployed by Best Festival Company in a safe and controlled environment that allows you to deploy contracts as if they were on a public blockchain.

from TryhackMe.

## Answers
1. What flag is found after attacking the provided EtherStore Contract: **`flag{411_ur_37h_15_m1n3}`**
