# CryptoGit Protocol

CryptoGit is a cryptocurrency that utilizes Bitcoin's proof of work system, but operates using Git as the blockchain backbone instead of a custom merkle tree structure like other popular cryptocurrencies. This Git-based blockchain is then centralized on GitHub.

This shows that much of Bitcoin's complexity already has existing, well-understood parallels in other technologies. Utilizing these applications to power a new cryptocurrency enables the same functionality but with less complexity and a less steep learning curve for people looking to understand the workings of a simple cryptocurrency.

## The basics

The _blockchain_ is represented by a Git repository hosted on GitHub.com. To mine on the blockchain, the repository can be cloned. To read information about balances or transactions, the repository can be cloned or viewed in the GitHub web UI.

A _block_ is represented by a Git commit on the `master` branch. Each block Git tree includes a few important files and folders:

1. `txn.json` is used to store a balance sheet mapping identities to balances.
2. `nonce` is modified during the mining process to manipulate the commit SHA.
3. `block_transactions/` stores information about transactions that have occured.

For a commit to form a valid block, it must have a SHA that is less than a pre-determined difficulty factor. This difficulty factor is published and updated hourly.

To add to the chain, a valid block must be pushed to GitHub as a branch off of master. A service consumes push hooks and validates new commits. If a commit is determined to be an acceptable new block, it is merged into master.

## Identities and authentication

In Bitcoin and other cryptocurrencies, public/private key pairs are used as a means of authentication and identification.

CryptoGit uses GitHub handles instead. `txn.json` is a map between GitHub handles and balances. E.g. `{"d12": 45, "nwoodthorpe": 56}`. Transactions are also handled using GitHub authentication instead of signing via a private key.

## Transactions

A transaction can be requested by creating a specially formatted issue in the blockchain repository the format is:

```
recipient,amount
```

`recipient` is the handle of the transaction recipient, and `amount` is how much currency to transfer. The sending user is automatically set to the GitHub user creating the issue. Since you cannot create an issue on behalf of another user, this handles transaction authentication.

Open issues in the repository represent open transaction requests waiting to be fulfilled by miners.

## Applying transactions as a miner

The list of open transaction requests can be fetched relatively easily using GraphQL:

```GraphQL
query {
  repository(owner: "d12", name: "git-blockchain-test"){
    issues(last: 100, states: [OPEN]){
      nodes{
        author{ login }
        body
        number
      }
    }
  }
}
```

For each transaction:

1. Verify the sender has enough currency to perform the transaction. Negative balances are illegal.
2. Apply the transaction to `txn.json`, adding `amount` to the recipients balance and removing `amount` from the senders balance.
3. Create a record of the transaction in `block_transactions/` with filename matching the issue number. 

The format of records in `block_transactions/` is:

```
sender,recipient,amount
```

Since GitHub issues are mutable, this ensures we have a full transaction history on the Blockchain that cannot be lost.

## Mining a new block

A new block can be mined by following these steps:

1. Pull `origin/master`
2. Create a new branch off of master. The name must be unique.
3. Apply transactions using the section above
4. Apply a 50 coin miner's reward to `txn.json`. This transaction does not need a `block_transactions/` record. The reward can be applied to any user, not just the mining user.
5. Start working through nonces until the commit SHA is less than the published difficulty factor.

Step 5 can be done using `git commit` and `git reset` repeatedly. A faster alternative would be to replicate the hash calculation functionality of `git commit`.

After a valid block has been mined, it can be pushed to a new branch on GitHub using `git push -u origin branch_name`. There is no guarentee that this block will be added to `master`, as another miner may have mined a block off of `master` first, which would make the newer block invalid as it's now based off of `master^`, not `master`.
