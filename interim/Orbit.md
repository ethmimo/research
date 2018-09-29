# E-Contracts and OrbitDB

Author: **Ghilia Weldesselasie** <br>
Date: *28/09/2018*

---

An interim solution for Mimo would be using OrbitDB in combination with a concept I call e-contracts.
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

*Can a malicious node bypass the code in a db and add random data?*
~~If a malicious node is capable of bypassing the db store's code then that completely invalidates the concept of an off-chain smart contract.~~
**Update 29/09/2018:** No peer is able to bypass the code in an orbit db store, which means that the code is law in our db.

*Can a node pinning our db be target and DDOS-ed, what happens if it is (one of) the only node pinning our db?*
~~If bringing down an orbit db is too easy then e-contracts will be too centralized and an easy attack vector to bring down Dapps.~~
**Update 29/09/2018:** Having a single node might be a risk factor but as more peers get on the network and with the advent of Filecoin, we can have those peers pin our db so that it stays censorship resistant.

These security concerns still need to be explored. The idea of a decentralized smart contract that works off-chain and for free is perhaps too good to be true, and needs to be thoroughly tested as blockchains have been in the past.

**Update 29/09/2018:**
## Issues with Consensus
There is a huge problem I had not considered concerning consensus on OrbitDB. Pubsub offers no guarantees on delivery speed and whether messages will be delivered to the rest of the network. Meaning if someone registers **@buidl** and then another peer tries to register the same name soon afterwards, the request might go through since it's possible the first registration has not yet reached the second peer.

This problem does not seem to be solvable unless we let go of the assumption that **@names** need to be unique. So in the example above, both registration of **@buidl** would go through.

*But then how would we differentiate between two profiles?**
If we introduce a system where we add tags to usernames, similarly to how Discord has id numbers attached to usernames, we could allow for multiple registrations of the same name. As long as the combination of tag and name is unique, then it can be differentiated which is fine for our purpose.

The tag can be generated from your Ethereum address upon registration, your address is encoded with [Daefen](https://github.com/alexvandesande/Daefen) and shortened for simplicity and better UX.

To recap:
- the `name` is your **@name**
- the `tag` is an identifier generated by your Ethereum account
- the `id` is the combination of your `name` and `tag`

In my case: my name would be `@ghiliweld`, my tag would be `jhoavfst` and my id `@ghiliweld:jhoavfst`.

Even if the tag isn't an actual word, it is still somewhat human readable which is all that's need to differentiate it between the several profiles who's names are **@ghiliweld** (in a dropdown selection for example). So if you just gave someone your name and an incomplete tag, they would *probably* be able to find the right profile.

An example:

- Me: "Hey can you send 0.5 ETH?"
- You: "Sure, what's your Mimo id?"
- Me: "It's @ghiliweld and I think the tag is 'jho...'. Not sure but it's something like that."
- You: "Hold up, lemme check. Ah, found it! The tag is 'jhoavfst'(pronounced) right?"
- Me: "Yup!"

**This is still very risky** and the tag system is still a work in progress, but this is the direction I'm considering in a system where we need some form of **uniqueness**, with **human readable identity** and **no consensus**.
