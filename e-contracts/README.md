# E-Contracts and OrbitDB

Author: **Ghilia Weldesselasie** <br>
Date: _28/09/2018_

---

An neat solution for Mimo would be using OrbitDB in combination with a concept I call e-contracts.
The result is an identity system that allows for gasless, off-chain and fast updates to one's profile.

## What is OrbitDB?
[OrbitDB](https://github.com/orbitdb/orbit-db) is a library for creating p2p databases on IPFS. They can be created for free and replicated among numerous peers. Changes on one peer are communicated to others via pubsub. Perfect for instant changes to any profile on Mimo. In this case, Dapps that want to implement Mimo would be the ones maintaining peers and replicating the Mimo DB.

OrbitDB offers several default stores like: key-value stores, logs and document stores. However, OrbitDB does offer support for Custom Stores. You can either extend the base `Store` class or extend one of the default stores like `KeyValueStore`.

This is an important feature to enable the use of e-contracts.

## What are E-Contracts?
E-Contracts are off-chain smart contracts that run on OrbitDB. They're called e-contracts because they don't require gas :)

Using Custom Stores on orbit, we can modify orbit DBs so that they require valid digital signatures to add data to the DB. For example, if we were to extend [`DocumentStore`](https://github.com/orbitdb/orbit-db-docstore) and use that as storage for profile information we would require that the owner of the profile sign data to be added. Now, `data` and  a valid `signature` have to be passed to `put()` in order for the data to be added. If the recovered signer of the signature is not the owner of the profile then the update is rejected. This works the same way across peers, essentially enabling an off-chain decentralized smart contract.

Here's a simple implementation of `transfer()` in the ERC20 standard done off-chain:
```js
transfer (cheque, sig) {
  /// Transfer an amount of token from one address to another
  //  cheque contains params: from, to, amount and a nonce

  // check is transfer was authorized
  if (recover(cheque, sig) != cheque.from) throw new Error('Transfer was not authorized.');
  if (typeof cheque.amount !== 'number' && !isFinite(cheque.amount)) throw new Error('amount is not a number or is not finite.');

  // code for nonces
  if (get(EthCrypto.hash.keccak256(cheque))) throw new Error('Transaction has already been done.');
  super.put(EthCrypto.hash.keccak256(cheque), true);

  // substracts amount from sender address
  put(cheque.from, cheque.amount, 'sub');
  // adds amount to destination address
  put(cheque.to, cheque.amount, 'add');
}

put (address, amount, operation) {
  if (operation == 'add') super.put(address, get(address) + amount);
  if (operation == 'sub') super.put(address, get(address) - amount);
  else throw new Error('Operation not valid');
}
```

This approach is:

- Free, there are no gas costs to update your profile since changes are made using signatures
- Fast, changes are propagated to other peers in a few milliseconds via IPFS pubsub
- Permissionless, any etherless account on Ethereum can create an identity in seconds
- Decentralized, data is on IPFS and someone's identity can't be deleted unless the owner wishes it.

Using OrbitDB, which is written in JavaScript, also has the benefit of taking advantage if the numerous open sources packages out there on npm. For example, using the `eth-crypto` npm package allows us to recover the signer of any signature passed through `put()`. It also allows for encryption which will be important for privacy in Mimo.

Using TypeScript would be better since it's statically typed but that's trivial enough to implement.

## Cons?
E-contracts are still highly experimental (and purely theory at this point!) and the fact that it's still built on immature technology (IPFS) is a risk.

IPFS' censorship resistant properties have not been thoroughly tested and it is still unknown if orbit dbs are as unstoppable as smart contracts on Ethereum.

_Can a malicious node bypass the code in a db and add random data?_
~~If a malicious node is capable of bypassing the db store's code then that completely invalidates the concept of an off-chain smart contract.~~ <br>
**Update 29/09/2018:** No peer is able to bypass the code in an orbit db store, which means that the code is law in our db.

_Can a node pinning our db be target and DDOS-ed, what happens if it is (one of) the only node pinning our db?_
~If bringing down an orbit db is too easy then e-contracts will be too centralized and an easy attack vector to bring down Dapps.~~ <br>
**Update 29/09/2018:** Having a single node might be a risk factor but as more peers get on the network and with the advent of Filecoin, we can have those peers pin our db so that it stays censorship resistant.

These security concerns still need to be explored. The idea of a decentralized smart contract that works off-chain and for free is perhaps too good to be true, and needs to be thoroughly tested as blockchains have been in the past.

**Update 29/09/2018:**
## Issues with Consensus
There is a huge problem I had not considered concerning consensus on OrbitDB. Pubsub offers no guarantees on delivery speed and whether messages will be delivered to the rest of the network. Meaning if someone registers **@buidl** and then another peer tries to register the same name soon afterwards, the request might go through since it's possible the first registration has not yet reached the second peer.

This problem does not seem to be solvable unless we let go of the assumption that **@names** need to be unique. So in the example above, both registration of **@buidl** would go through.

**But then how would we differentiate between two profiles?**

~~If we introduce a system where we add tags to usernames, similarly to how Discord has id numbers attached to usernames, we could allow for multiple registrations of the same name. As long as the combination of tag and name is unique, then it can be differentiated which is fine for our purpose.~~
**Update 04/10/2018:** I'm ditching the tag approach since it makes for terrible UX. Users would have to remember both their names and their generated tag (which they don't get to choose). Check out the update on Oct. 4 to see what's next.

We need some form of _uniqueness_, with _human readable identity_ and _no consensus_.

**Update 04/10/2018:**
See [Cosigning.md](Cosigning.md) for research on unique, human readable names without consensus.
