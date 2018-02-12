# cryptogit-development
This repo is meant as a place for tracking development of Cryptogit. Issues, documentation, and code relevant to Cryptogit will be stored here. This is not the blockchain repository.

## What is CryptoGit?

CryptoGit is a cryptocurrency that uses Git as the blockchain backbone instead of a custom merkle tree structure like other cryptocurrencies. CryptoGit is centralized on GitHub.com

## How does it work?

The blockchain is represented by a GitHub repository. To blocks are represented by commits to the master branch. If a valid commit is created branching off of master, it is auto-merged into master by a service listening for push hooks.

A commit is valid if:
1. It branches off of the latest master
2. It has a valid commit SHA. That is, the commit SHA must be less than the dynamically calculated difficulty factor. This difficulty factor adjusts as computing power joins the network to keep the frequency of new blocks roughly constant.
3. It has valid transactions. This includes a block-reward to the miner, plus user -> user transactions.
4. No other files have been added, deleted, or modified.

If a commit does not satisfy any of the above criteria, it will simply sit as a branch off of master, and soon get outdated as master moves forwards.

## Authentication and User Addresses

Bitcoin and other cryptocurrencies use public/private keys as a means of identification and authentication. You can identify a user by their public key, and make transactions on behalf of a user using the private key. CryptoGit does not work like this.

A user is identified by their GitHub handle. Transactions are addressed to/from GitHub users, and balances are assigned to GitHub users. 

## User -> User Transactions

To initiate a user -> user transaction, an _Issue_ is created in the cryptocurrency repository using a special format. This issue includes what user to send currency to, and what the amount is. Notice we have no need for private key signatures since an issue has an author, and GitHub authentication prevents you from creating an issue as another user.

Miners can fetch requested transactions by querying for open issues in the Cryptocurrency repository. Notice this can be done with a single GraphQL query.

Once a miner has a list of transactions it would like to add to it's block, it adds some information about each transaction to the commit (exact format specified in the formal protocol), and applies each transaction to the balance sheet. This transaction information and balance sheet will be verified by the verification server before a block is merged to master.

## So it's not decentralized, and it relies on microservices to operate. Doesn't this defeat the purpose of Bitcoin?
Yes.
