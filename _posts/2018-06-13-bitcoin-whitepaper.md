Bitcoin

# p2p timestamp system

1. chronological cryptography vs trust on 3rd party institution;
2. solve double-spending issue;
3. depends on the fact that honest nodes collectively outpace attacker nodes;
4. proof-of-work;

# transaction

1. we mine a block (and get coin reward) istead of a coin; bitcoins are created each time a user discovers a new block.
2. coin is defined as a set of transactions digitally signed; payee can verify the payer and cannot verify whether it's the earliest transaction, thus cannot exclude double-spending;
3. all transactions are publicly announced;
4. a transaction has inputs (several) and outputs (at most 2). Inputs are existing outputs from other transactions for you. Hence, transaction actually takes outputs for you and then generate new outputs, one of which is for payee and the other is for change. The remaining (sum of inputs minus sum of outputs) is Bitcoin fee.
5. we need 'change' because outputs cannot be partially spent. You can only spend your output as a whole, which explains why we could have 'change' output.
6. wallet (account balance) of a peer (identified by private key) is derived from all transaction outputs for his public key, namely 'pay-to-pubkey-hash' transactions.
7. To get the concept of transactoin, read https://bitcoin.stackexchange.com/a/20623 and https://bitcoin.stackexchange.com/q/736

block reward = block subsidy + transactions fees

# timestamp server

1. another hash on the whole transaction block;
2. each item in the block consists of a transaction and timestamp thereof; transaction items in a block are usually unrelated.
3. payee can verify it is the earliest and not a double-spending transaction. kick out double-spending transaction.
4. announce the hash and timestamp together.

# proof-of-work

1. how to hash? simple sha-256 is not enough! Add nonce to find a special hash value that starts with a number of zero bits. The number is increased overtime (more and more difficult).
2. proof-of-work is where our 'mining resources' are invested and represented by 'minining resources' invested.
3. proof-of-work is deliberately designed to avoid double-spending such that attacker cannot catch up.
4. Nodes compete to confirm a transaction, namely compete to tally!

# p2p networking process

1. new transaction is simply hashed and signed before announcement.
2. a block (local copy) comprises a set of transaction items, previous hash, nounce. received transaction along with timestamp are inserted into the transaction set.
3. difficult hash computational work for block, namely proof-of-work.
4. success! a peer announces hash and the block.
5. other peers verify and accept the block.
6. other peers create a new block by replacing the 'Prev Hash' field and announce it.

Conceptually, there is only one valid (of course longest) blockchain over the network. But techniquely, there exist mutiple blockchains simutaneouly due to node joining/leaving, network latency etc.

# double-spending

1. Timestamp is acutally not critical element since attacker node can forge local time. Instead, the longest blockchain is accepted while other candidates are discarded.
2. Currently, user confirm a transaction if it buried over 6 (included) blocks. 6 blockes are enough to avoid catching up by powerful attacker. If an attacker double-spends coins by signing second transaction immediately (say one to Alice and another to Bob), only one of them (say that to Bob) is accepted when it is buried in a longer blockchain.
3. It is not really the Proof of Work which prevents double spends but rather the blockchain itself which prevents double spends. The Proof of Work is just one aspect of the blockchain.

https://bitcoin.stackexchange.com/q/61385

# incentive

1. the first transaction in a block is a special transaction that starts a new coin owned by the creator of the block; the transaction set is empty.
2. transaction fees; fee is given to the peer that achieve 'proof-of-work' - the one that find a block. Bitcoin fee is fixed independently of the amount of coins.
3. attacker with more mining resources probably chooses to be honest as that would earn more coins than stealing.

# privacy

1. peer public key is anonymous though transactions are public.

# Economy

1. Bitcoin amount limit would fail in *deflation spiral*. But that is not a problem as Bitcoin cannot be global currency.
2. Currently, miners mine bitcoins assuming price increase in the future.
3. People use bitcoins to bypass censorship.

# 中文

1. 账本，区块链，blockchain;
2. 矿池，asic hwardware node;
3. 挖矿，mine, proof-of-work, hash with leading zeros;

# todos

1. asic hardware;
2. testnet for fun;
3. p2p timestamp? what if fake node time?
4. centralized Bitcoin exchange?
5. how many bits of a hash is required?

# reference

1. Bitcoin: A Peer-to-Peer Electronic Cash System
2. https://en.bitcoin.it/wiki/Controlled_supply
3. https://bitcoin.org/en/how-it-works
4. https://bitcoin.org/en/you-need-to-know
