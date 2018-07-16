# XCredits-Style Private, Off-Chain Transactions (XSPOCT)
Author: Dr James Jansson, Lead Researcher, CEO, XCredits

**NOTE: this is NOT the XCredits Core whitepaper. It is simply a small part of the XCredits protocol, which can be applied to the Bitcoin protocol, and as such we have decided to release early. Please visit the [Contents](https://github.com/XCredits/xcredits-whitepapers/blob/master/README.md) to see the status of that paper.**

**WARNING: The protocol specified in the below paper is in DRAFT only. The protocol may change in the future. Given that this protocol may change, and the process requires destruction of Bitcoin to create XSPOCT, you should only convert and receive amounts that you are willing lose or experiment with.**


# Abstract
In this paper we describe a new methodology for making private transactions in cryptocurrencies. 

Bitcoin and many other cryptocurrencies provide certainty through a public blockchain. Unfortunately, this leads to a loss of privacy, which may slow adoption of cryptocurrencies. 

The XSPOCT technology can work with most established cryptocurrencies, including Bitcoin, Ethereum and other blockchains. XSPOCTs do not require a separate sidechain, and instead use the confirmation power of the underlying blockchain, removing the need for timelocks.

XSPOCTs allow (largely) private transactions in a hostile network between two people, either of whom may be blocked by the network ordinarily or who may be the target of tracking. 

# Important links
Demonstration code for this paper can found in the [XCredits XSPOCT Repo](https://github.com/xcredits/xspoct).

This paper may go through some changes. You can find this paper, and its versions at [our GitHub repository.](https://github.com/XCredits/xcredits-whitepapers/blob/master/xspoct.md). 


# Background
## Cryptos are often not very private
Platforms like Bitcoin and Ethereum are considered secure because the transactions that are made on them are made in the clear on a public blockchain. All the inputs and outputs are visible to all participants in the network. 

## Existing methods for privacy
There are a number of privacy-focused coins such as Monero, Dash, ZCash, Verge and PIVX Digital Note. I won't go into a lot of depth of these platforms here, but have more discussion about some platforms in the section ["Summary of some other privacy methods"](#Summary-of-some-other-privacy-methods)

The main two methods of privacy are mixing/tumbling and Zero Knowledge Proof methods, both of which have advantages and limitations. In short, mixers are attractive to use as honeypots by nation-state surveillance and Zero Knowledge Proof Methods are very hard and unproven in a sharding situation. 

## Cryptocurrency is valuable for three reasons:

1) It is hard to make more coins
2) It is tradeable
3) It is programmable 

The immediate intention of XSPOCTS is to preserve features 1 and 2. To demonstrate it is hard to make, we'll be taking an established coin (Bitcoin), converting it to an "XCredits-Style Private, Off-Chain Transaction" Note (XSPOCT Note), and establishing a system for trading these coins. 

## Proof of non-existence: the most important invention of Blockchain
Proving that something has happened is relatively easy. All you need to do to prove that transaction A happened prior to a particular time is to ensure a hash of the data contained in transaction A is included in some future transaction B. 

**The real value of blockchain is being able to query the blockchain and demonstrate that something is not in the blockchain.**

Proving order of non-hash-chained events (i.e. that transaction A happened before transaction B) is simply a derivative of proof of non-existence (proving that transaction B did not exist before transaction A).

## Bitcoin's transparency is overkill for the majority of simple transactions
Programmable transactions through Bitcoin's Script and smart contracts through Ethereum's Solidity are both very useful. Unfortunately, both also require the transaction details (include the script and the details of who sends and receives coins) to be sent in the clear in order to operate on the transactions.

Meanwhile, the majority of value exchanges people intend to make have limited contractual or multi-party elements to it. In almost all instances, people just want to swap one asset (currency) for another (different currency, good or service).

As such, the trade-off of privacy for smart contract capability (which the end-user is not using) is an unacceptable one, especially for people concerned about privacy in the wake of major privacy scandals, people who may be agitating against oppressive regimes and people who are purchasing products that are socially taboo.

## XSPOCT - provable yet private transactions
XSPOCT makes transactions private by ensuring that the proof that a transaction output has been spent in a provable, reliable way on the blockchain, while keeping the details of the transaction hidden from view. 

# Method

In this example, we have Alice who currently has some Bitcoins, Alice's "Buddy", some as-yet-known person who wants to receive some XSPOCT, and "Comrade", some as-yet-known person who "Buddy" will do a transaction with in the future.

## 1 Creating XSPOCTs from Bitcoin
### 1.1 Alice generates Spend Info Outputs
In the Spend Info Output, there are two secrets:

1. **Address Secret:** A private-key/public-key/address, which is the place where Bob will tell Comrade to look for proof he sent them funds in the future.
2. **Hidden Secret:** A private-key/public-key, which will form part of the message of the transaction that Bob's future "Comrade" will decypher in order to determine that Bob sent "Comrade" the XSPOCT first.

Alice makes sure to hold on to the private key corresponding to the Hidden Public Key, as this is the only way she will be able to spend her XSPOCT. She may also choose to keep the Address Secret's private key, as she will have to send a transaction to this address on the blockchain to prove she is moving the XSPOCT and she may want to recover the funds.

## 1.2 Alice Creates a string of the Address Secret and public key of the Hidden Secret
~~~
SpendInfoString = {
  "address":  "mn189...",
  "hiddenPublicKey": "a1b8...",
  "specification": "0.1",
  "verificationPlatform": "bitcoin-testnet", // where the next transaction will be found
}
~~~

## 1.3 Alice hashes the Spend Info Strings
There will be multiple Spend String Outputs included in this burner transaction. Alice doesn't want other people to know where she will later spend the XSPOCT. By combining the address and the hidden public secret, the resultant hashes cannot be used to find the future transactions on the blockchain.

## 1.4 Alice generates a Burn Transaction String

~~~
burnTransactionString = '{
    "source": utxo,
    "isBurn": true,
    "amount": "0.02"
    "outputs": outputDetails,
    "specification": "0.1",
    "originalPlatform": 'bitcoin-testnet', // where the transaction came from
  };
}'
~~~

There are two parts unspecified above, `utxo` and `outputDetails` in the above string, which we will expand below.

`utxo` is the unspent transaction output from which the Bitcoin burn will occur.
~~~
utxo = { 
    "txId": "58e9829...",
    "outputIndex": 0,
    "address": "mnMTa...",
    "script": "76a91...",
    "amount": "0.55000000"
}
~~~

`outputDetails` contains the details of the way that the XSPOCT can be spent in the future. It is an array of the spendInfoHashes and the amount that each hash should receive.

~~~
outputDetails = [ 
    { 
      "spendInfoHash": "7c10f...",
      "amount": "0.01"
    }, { 
      "spendInfoHash": "be5e6....",
      "amount": "0.01"
    ]
~~~

It is very important that each hash contains a public key from a brand new address, as the first correctly signed result will be considered the transaction that actually occurred.

## 1.5 Burn some existing Bitcoin
We'll be sending real Bitcoin to an unspendable address.
~~~
!! SENDING BITCOIN IN THIS WAY RENDERS THE BITCOIN UNRECOVERABLE TO A REGULAR BITCOIN!!
~~~

Double SHA256 hash the above burnTransactionString. This will become the "Public Key" where we are sending the coins. Generate an address from the public key above, then send the specified amount of Bitcoin to that address.

I use the quotes for "public key" because it is not a real public key. There is no way to derive the private key from this fake public key. This is important, because we need to be able to prove that this Bitcoin cannot be sent elsewhere.

## We have now successfully created XSPOCT from Bitcoin
The next step is to send the XSPOCT to someone.

## 2 Sending the XSPOCT from Alice to Alice's "buddy", Bob

### 2.1 Bob Creates a string of the address and public key of the hidden secret
As Alice did in 1.2, Bob will create his Spend Info Strings. In this example, Bob will not divide the outputs into smaller amounts, so he will only send a single Spend Info String.


## 2.2 Bob hashes the spend info string, then sends it to Alice
Bob doesn't want Alice to know where he will later spend the XSPOCT, so he sends her the hash of the Spend Info String.

## 2.3 Alice generates a XSPOCT String
~~~
xspoctString = '{
    "source": "7a8b...", // Hash of previous (or burn) transaction string
    "amount": "0.01"
    "outputs": [
      {
        "address":  "ma195...",
        "hiddenPublicKey": "9c10...",
        "specification": "0.1",
        "verificationPlatform": "bitcoin-testnet"
      }
    ],
    "originalPlatform": 'bitcoin-testnet',
    "verificationPlatform": 'bitcoin-testnet'
  };
}'
~~~


## 2.4 Alice sends a small Proof Transaction to an address that is included in her Spend Info String

Alice created an address in the Spend Info Strings in 1.2. She will send a Proof Transaction on the blockchain to an address in one of the Spend Info Strings to show that she wants to send that XSPOCT to Bob.

In the Bitcoin transaction description, she includes a string in the following format:
~~~
hiddenPublicKeyHash + xspoctStringHash + signature
~~~

where 
 
- hiddenPublicKeyHash corresponds to the hidden public key Alice generated in 1.2.

- xspoctStringHash is from 2.3

- signature is derived from the hiddenPublicKey's private key generated in 1.2


### Censorship resistance
Externally, this looks just like random noise in hex format, like a collection of three hashes joined together. It is hoped that this will increase the censorship resistance of XSPOCTs

The hiddenPublic key plays a very important role. When we send the transaction to the blockchain, it is possible a person who dislikes the idea of XSPOCTs may be listening, and looking for transactions that look like XSPOCTs. If they can send a bigger transaction to the address specified first, then they would be first on the blockchain with that address. As such, we cannot simply send a transaction to a secret address as proof. We need to include a message that includes a secret that only the receiver of the XSPOCT will know, but the sender can't lie about.

Note that the system could have been designed such that the private key corresponding to the Bitcoin address signs the xspoctStringHash, such that the description is of the form: 
~~~
xspoctStringHash + signature
~~~
However, the nodes on the Bitcoin network would be able to tell that the first part of the message was signed by the receiving address, making it possible that the nodes may reject the transactions.

We cannot hash the signature, because Bob needs to be able to prove that the signature corresponds to the hashed public key at the beginning. If we hashed the public key (i.e. the message looks like `hiddenPublicKeyHash + xspoctStringHash + hashOfSignature`), a person who wanted to prevent XSPOCTs would be able to send a transactions that looks like:
~~~
hiddenPublicKeyHash + fakeRandomCharactersXspoctStringHash + fakeRandomCharactersSignature
~~~

Neither Bob, nor Alice, would be able to prove that the string wasn't a real transaction signed by the public key. 

## 2.5 Alice sends the details of the transaction and the burn information to Bob. 

Alice needs to send the data of the full chain of the transactions to Bob in order for Bob to determine whether the coin is derived from a real coin or not, and whether the coin has been spent elsewhere.

## 2.6 Bob checks on the blockchain that this process has occurred
1) Bob hashes the Burn Transaction String, then generates an address from that hash. Bob then checks that the address hash received the real Bitcoins specified.
2) Bob looks up the address that is specified in the address in the spendInfoString
3) Bob checks that the message conforms to the format
`hiddenPublicKeyHash + xspoctStringHash + signature` by checking:

- that the first 64 characters is the hash of the `hiddenPublicKey` 
- that the second 64 characters is the hash of the `spendInfoString`
- that the third 64 characters is the signature of the `spendInfoString` signed by the `hiddenPublicKey`

## Bob has now confidently received a XSPOCT
He can now send it to his Comrade, Charlie. 

# Sending XSPOCTs from Bob to Charlie
The methodology that Bob followed in section 2 is now followed by Charlie, and while Bob now does what Alice did in 2.

1. Charlie generates Spend Info Strings as Bob did in 2.1
2. Charlie sends the hashes to Bob as Bob sent to Alice in 2.2
3. Bob generates an XSPOCT string as Alice did in 2.3
4. Bob sends a small Proof Transaction (like Alice did in 2.4) to an address that is included in his Spend Info String (that he generated in 2.1)
5. Bob sends all the transaction details, from the burn transaction string Alice created, to the XSPOCT transaction string that Alice did to transfer to Bob, and finally the XSPOCT transaction string that Bob did to transfer to Charlie
6. Charlie checks the original burn transaction like Bob did in 2.6, and all the subsequent XSPOCT transactions.


## General comments about the methodology
### Single-use private key
The XSPOCT Note method effectively forces single-use private/public key pairs, because the existence of a public key signed transaction on the blockchain indicates that a transaction involving that key has occurred.

### Proving that a transaction has resulted in a XSPOCT.
It may be desirable to prove that the coins in question are no longer available to the operating supply of the underlying coin.

The double SHA256 is used to allow it to be proven that a transaction is an unspendable address without revealing the contents of the output details.


# XSPOCT Notes 
## Choice of name
I have chosen the name "XSPOCT Notes" to represent these tradable assets.

Firstly, I like the term "note" because XSPOCT Notes are similar to notes in the real world. If Bitcoin or other cryptocurrencies are the limited coin asset, then XSPOCT Notes are simply a way of representing that. In main countries, paper notes were once redeemable for the underlying metal backing the currency in coin form. Today, those notes are not redeemable for the underlying metal coin. Once you create an XSPOCT Note from a Bitcoin or other coin transaction, you cannot convert the XSPOCT Note back into usable Bitcoin.

The second reason why I like the idea of calling it a "note" is that in order to store and send the notes, the owner must keep a local record of all the transactions associated with the note. 

Finally, like real life notes, it shouldn't be divisible like a gold coin is. A lot of banks will accept parts of notes based on the size of the piece you have. However, it's not advisable to break your notes into fractions. In the same way you can, but probably shouldn't, break up your XSPOCT Notes if you want to remain anonymous in the future. See [Standardized Note Face-value](#Standardized-Note-Face-value).

## Standardized Note Face-value
Although it may be tempting to have arbitrary transaction values associated with a XSPOCT Note, very unique transaction values can act as an identifier for particular transactions and hence increase traceability both for transactions upstream and transactions downstream. 

Rather, in a single transaction, multiple notes should be transferred. This does use more transaction capacity of the chain and does increase fees, however the aim is to be more private, and privacy has a cost associated with it. 

## Avoid note joins
Although it is theoretically possible to do a note join by including 2 unspent XSPOCT Note transactions, it is highly unadvised. This is because it unnecessarily joins two unrelated notes into a specific transaction, possibly revealing more information about the spender and the people upstream of the payment. It is better, as said above, to do multiple on-chain confirmations than join notes into a single transaction. 

## Be conscious of linking XSPOCT Notes from different transactions






# Desirable blockchain platform qualities 
To ensure the best performance and privacy of XSPOCT Notes on a blockchain, certain blockchain qualities would ideally be present. 

## Supports lots of transactions
Above we discussed standardised transaction sizes as a method of maintaining privacy. When we use standardised note sizes, the number of transaction outputs that are necessary to complete a transaction could go up significantly. 

For example, to spend 0.83 XSPOCT Bitcoins, we might need to make a transfer of 0.5BTC, three transfers of 0.1 BTC and three transfers of 0.01 BTC. 

At just 3 transactions per second, the above 0.83BTC transaction would take 2 seconds of total network time. This means that it is desirable for a platform to have a large number of transactions per second.

## Cheap or free transactions
If we are doing lots of transactions, say 4 or 5 per transaction on average, fees are going to be 4 or 5 times as high. Obviously, the cheaper that fees are, the easier it will be to transact in this private way.

## Easily mineable
Ultimately, all the on-chain transactions must be paid for on-chain, either by fees or with a small dust transaction. The coin used to pay for the transaction has to come from somewhere, and ultimately could reveal to downstream holders of the XSPOCT note the identity of the people above them. 

To get even more anonymized transactions, the funds used to pay the fees for the on-chain transaction would be derived by anonymously mined coin. In this way, there would be not a method of connecting the fee for the transaction to the transaction itself.

As such, an ideal platform would have a mining reward that would pay out in Reasonable Time™. By Reasonable Time™, I mean that a standard, consumer grade computer would get a mining reward on average after a week of full-time mining. 

Under current mining conditions in Bitcoin, the block reward is too high to be practical for the average user to generate new coins to validate their XSPOCT Note transactions. In June 2018 prices, `12.5 Bitcoins per block * US$6500 per Bitcoin = US$81,250 per block`. With this assumption, and that an average computer might generate 10 cents worth of value per hour, the user would have to leave their computer on for 812,500 hours, or 92 years. This is way too long for an average person to generate a mining reward in order to make their transactions private.


### Note on the lower limit of mining time
Given that the mining reward would be permanently entered into the blockchain, the amount of mining necessary cannot be the mining reward for a single transaction. A minimum estimate of the size of the block reward is 100 transaction fees' worth, however it might be higher than that again. A blockchain solution that aims to accommodate XSPOCT Notes at maximum privacy levels should aim to minimise transaction fees as much as possible, as well as avoid aggregating too many transactions into a single block.




# Limitations
## Must be on chain - scaling limitation
The on-chain confirmation must *really* be an on-chain transaction. As such, XSPOCTs cannot occur on lightning networks. All transactions are only confirmed when they have reached a level of certainty on the blockchain that is acceptable to the receiver.


## Once destroyed, the Bitcoin can never be recovered
The Bitcoin is sent to a burn address in order to prove that the XSPOCT Note representing the Bitcoin (or other underlying cryptocurrency) cannot be converted back into Bitcoin that is usable on the Bitcoin network. This will continue to be the case unless a major change in the Bitcoin protocol occurs. 


## Protocol, once established, can never be changed for historical transactions
The spec for the method of transfer must remain the same for historical transfers to avoid conflicting outcomes. For example, changes to the method for identifying on-chain confirmation transactions may allow for double spending of XSPOCT Notes.

It is therefore a good idea to include with the transaction a hash of the spec that is used to determine how transactions are made. It is possible to change the spec for transmission downstream, however the change in spec must happen after a spend instruction is written to the blockchain, not at the same time or prior to the spec change being included.

The process would happen as follows:
1) XSPOCT Note holder wishes to change the protocol for on-chain confirmation. They find the hash of the protocol they want their XSPOCT to follow, and generate spend hash which includes that spec change. 
2) Perform an on-chain transaction confirming this new transaction pubic key and specification. 
3) Once this has occurred, the current holder of the coin can confirm future transactions using the new specification.

# Privacy issues 
Surveillance of transactions is not impossible in this system, simply much harder than in a purely open system. The core limitation of XSPOCT Notes is that all prior transactions are displayed to the receiver of a payment. 

Once a XSPOCT Note leaves your hands, you do not have any idea where it is. You cannot tell if it has been moved by the person you gave it to or not, which is a substantial increase in privacy for the receiver over Bitcoin. In Bitcoin, the recipient may be invisible prior to sending (because the public key is hashed). However, once the receiver moves the Bitcoin, the transaction time, value, new recipient and message data are all visible to the entire network.

The issue with the receiver knowing all previous transactions is that they hold a certain amount of power over the people who previously did transactions on the XSPOCT Note. 

## Fees and anonymity
You must pay for transaction fees using the cryptocurrency transaction currency. This is a potential vector for discoverability, as the fees paid for the transaction can link XSPOCT Note transactions in the event that downstream recipients of this payment reveal the contents of the XSPOCT Note. 

Trying to obtain fee-free transactions is a way to reduce identifiability, however, you still need to transfer real Bitcoin in order for the transaction to complete (and not be seen as a spam transaction). 

Once this solution becomes popular, it is likely that services will be established that accept small XSPOCT Notes in exchange for validating both the small note that pays for the transfer and the larger note of the intended transaction. 

Ideally, the ability to do a small amount of mining and be rewarded for the transaction will harden the algorithm against privacy attacks. XCredits Core will have a solution that addresses this issue.

## Nation-state surveillance through coin cycling
Nation states may collect taxes in XSPOCT Notes or offer services that may mean that XSPOCT Notes end up observed by nation states. Hence there is a general necessity to anonymize transactions.

## Nation-state surveillance through mandatory auditing
If a large enough nation state or collection of nation states decides that it wants to monitor all financial transactions, they potentially can do so by requiring all payments be audited under law. Many people who use cryptocurrency believe that the state has no power over them. But if a lifetime in jail is on the cards, most people will do what is asked of them. 

While I believe that this is an issue that could affect people's rights, it is not a deal-breaker, mainly because the four current options are:

1) Traditional banking systems that could easily be required to do full auditing of digital transactions. 
2) Current non-private blockchain solutions like Bitcoin which send all transactions in the clear and allow all people to see all transaction sources, destinations, amounts and messages. 
3) Mixers, which are a honey pot for national security agents to try to make up the majority of transactions to establish inputs and outputs, and can do so at a low cost
4) XSPOCT Notes that aren't perfectly resistant to nation-state surveillance but are better than in-the-clear Bitcoin transactions, but most importantly resistant against small to medium corporations and private individuals snooping. 

By far, XSPOCT Notes are more likely to afford the users general anonymity.

### Potentially a desirable level of auditability compatible with historical western requirements and rights
For your own transactions, you have a record of who you send to and from, as well as a historical track record of all the transactions prior to you on the XSPOCT Note. When compared to cash, XSPOCT Notes have increased record keeping.

If the government obtains a warrant for the financial information of an individual (and accesses the individual's computer), they can see all the transactions of the individual. However, they would be required to get a warrant of those people to dig deeper into a case. 

This method of information access and policing is akin to the assumption of innocence and the pretence of privacy that western nations, up until recently, held as a virtuous quality.

## Dox threat
One potential issue is that people who are downstream of the original payment can release full information about the string of payments. It is for this reason that the payment of fees should be as anonymous as possible. There isn't a great way to avoid this release of information, however, the release of information would have limited effect, as the majority of transactions would remain private. 

## Potential methods of avoiding privacy limitations
One way to reduce privacy issues is to use a larger third party to receive, store and swap your XSPOCT, all of which is possible without revealing to the person who gave you the XSPOCT Note that you have swapped it into a mixing organisation. Now, this organisation does not have to be an illegal or unregulated institution. In fact, it may be a well-regulated institution such as a bank.

### Example method
Below I use the term 'bank', however, the 'bank' in this case is any organisation with sufficient transaction capacity to help anonymize XSPOCT transactions.

Steps:
1) User structures transaction
2) User sends transaction to their bank
3) Bank send the transaction to the chain

The fees are paid for by the bank, who effectively hides who paid for a transaction. This is similar to a transaction mixer, but would be legal because the bank would have done KYC on the sender. 

### But why bother with cryptocurrency through a bank?

1) I believe that most people would be happy to go to a bank to purchase cryptocurrency. In fact, I'd suspect that many people would be prefer going to a well-regulated financial institution to purchase cryptocurrency, than to do so through poorly regulated exchanges or from strangers in face-to-face situations.

2) Banking, in the future, may have a different service offering to what it does currently. With widespread adoption of XSPOCTs, the banking industry may instead focus on privacy management and blockchain facilitation as a service.

Although largely impartial, banks can and have taken positions on things that their customers can and cannot buy, trade and borrow against, or do as part of their business, even in situations where the item or activity is completely legal. I understand the hostility, however broader adoption of cryptocurrency by banks will lead to increased auditability of money creation, as well as a greater emphasis on privacy.

# Summary of some other privacy methods

This list is far from exhaustive, but is intended to explore the limitations of core suggested solutions.

## Mixers/Tumblers
There have also been attempts at privacy such as mixing/tumbling schemes in CoinJoin, however these can be a honeypot for nation-state surveillance, where surveillance organisations can heavily populate attempted private transactions and easily join the senders to the receivers. 

An example includes Monero which uses ring signatures (mixers) with Confidential Transactions ([discussion about Confidential Transactions from the Elements Project](https://elementsproject.org/elements/confidential-transactions/)) to protect the values going in and out. 

## Zero Knowledge Proof systems
Zero Knowledge Proof systems use encryption methods that preserve information about the transaction, but do not reveal that information when transacting. Instead, the Zero Knowledge Proof methods allow a person to demonstrate that they know about a transaction output that is available, and that it hasn't been spent yet, and that the amount that is being sent is the correct amount and not fraudulently obtained, all without revealing which is the true transaction it has access to out of the (potentially many) original input transactions (mint transactions) and subsequent transfer transactions it references when hiding transaction details. 

The best example of this is ZeroCoin/ZeroCash/ZCash implementations. ZeroCash protocol was implementable on top of Bitcoin (in a similar way to XSPOCT Notes), but the project is focused now on a standalone implementation on ZCash. [Alessandro Chiesa’s talk](https://www.youtube.com/watch?v=uEdx-pqJg4I) on this demonstrates the motivations for privacy, a handful of approaches to implement private transactions and a description of the method of ZKSNARKs. (Aside XSPOCT Notes are similar to "Attempt #2" in this presentation, but the serial number reveal is put on the blockchain in hashed form, meaning that only downstream receivers of the note know about the transaction.) 

### Limitations of Zero Knowledge Proof systems
ZeroCash requires its own blockchain to manage the transactions (mint and pour transactions).

Most importantly, ZeroCash protocol requires knowledge of other ZeroCash transactions to ensure proof while maintaining anonymity. In the implementation in the ZCash blockchain system, this is not an issue because all transactions occur using ZCash mint and pour transactions. However, on blockchains like Bitcoin, you need to identify transactions that have qualities of ZeroCoin. This leaves them open to being censored by the network. It also limits the amount of anonymity provided to ZCash holders to the number of mint transactions demonstrated on the Bitcoin network. 

By contrast, XSPOCT Notes can be created and traded immediately, with only a single sender and receiver necessary to participate in such a system. That means that no critical mass is necessary, as is required with ZeroCash.

ZKSNARKs also have not been demonstrated in the context of sharding, and it is quite possible that sharding cannot occur while maintaining a system that is resistant to double spending under a ZKSNARK system. 

Another issue is that the ZCash Ceremony requires trust in the founders to set the basis for all future transactions. Vitalik Buterin has [an explanation of the Trusted Setup system](https://medium.com/@VitalikButerin/zk-snarks-under-the-hood-b33151a013f6) used in the ZCash Ceremony if you want to look at the technical details. While I personally don't believe they have lied to anyone about the creation of private keys for the system, it is not externally provable that the private keys generated have been destroyed. 



