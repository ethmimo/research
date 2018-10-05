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
~~If a malicious node is capable of bypassing the db store's code then that completely invalidates the concept of an off-chain smart contract.~~ <br>
**Update 29/09/2018:** No peer is able to bypass the code in an orbit db store, which means that the code is law in our db.

*Can a node pinning our db be target and DDOS-ed, what happens if it is (one of) the only node pinning our db?*
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

We need some form of *uniqueness*, with *human readable identity* and *no consensus*.


**Update 04/10/2018:**
## Cosigning Name Registrations
Consensus on IPFS doesn't seem possible to do for free and quickly among a huge networks of peers (hence why blockchains are even a thing). So if we can't have consensus among all our peers then we need a single source of truth concerning name registrations.

**But then how do we use a single source of truth without dealing with the cons of centralization (single attack surface, lack of transparency/trustlessness)?**

Turns we can do that with OrbitDB too :)

**Using OrbitDB**, we can have a decentralized **cosigner**, an impartial claim signer in our system, sign claims of name registration. Our cosigner will also add a timestamp claims it signs, so that when parties need to verify who made a claim on a name first they can refer to our cosigner.

Here's a simple implementation of our cosigner on orbit:

```js

const EthCrypto = require('eth-crypto');
const Store = require('orbit-db-store');

class Cosigner extends Store {
    constructor() {
        // A keypair is generated for our cosigner
        const _details = EthCrypto.createIdentity();

        // We create a getter to allow access to our cosigner's pubkey
        this.publicKey = function() { return _details.publicKey; }

        // We create a function that cosigns claims with our cosigner's privkey
        this.signClaim = function(registration) {
          // We get a prepped claim with a timestamp
          const claim = prepClaim(registration);
          const signedClaim = EthCrypto.sign(_details.privateKey, EthCrypto.hash.keccak256(JSON.stringify(claim))
          );
          // We return both the original claim and the signed claim
          return { claim, signedClaim };
        }
    }

    prepClaim (registration) {
      if(registration.owner != recover(registration.name, registration.signature)) throw new Error();
      // A timestamp is generated for when our claim was made
      const timestamp = new Date();
      // We return a prepped claim with a timestamp
      return { registration.name, registration.owner, timestamp };
    }
}
```
Let's run through what's going on here.

First, we submit a `registration` to the `signClaim()` method. A `registration` should include:
- a `name`
- an `owner`, the address that is claiming ownership of the name
- a `signature`, the `name` signed by the `owner`'s private key.

Next, our `registration` is prepped by the `prepClaim()` method. This method simply removes the `signature` and adds a `date` (a unix timestamp) to our claim.

Finally, our claim is passed to `signClaim()` and signed by the cosigner's private key and returned with the prepped claim.

## Why cosigning on Orbit works
In a system where achieving network wide consensus is either too expensive or too inefficient, we have no choice but to leverage centralization to validate registration claims.

However, with centralization comes the risks of shutdown since centralized entities are easier and more profitable targets to go after.

OrbitDB is the key to an uncensorable/unstoppable single source of truth on IPFS. By distributing our cosigner on IPFS via Orbit we can have a cosigner that stays up as long as it's pinned by a peer and works the same for all peers.

This cosigner approach allows to verify who claimed a name first and award ownership of a name to the it's rightful owner (the first person to claim it). This is verified through the timestamp we attach to claims.

Meanning if we make a claim on a name before anyone else in the world, but because of issues with consensus on pubsub someone else takes ownership of a name before us, if we can prove that we really did claim this name first (with our timestamp) then can take ownership of the name at any time. This is where our impartial cosigner comes in to back up our claim. Alternatively, no one will be able to take our name from if we're the first ones to claim it.

Let's call this **Proof of Dibs (PoD)** :)

## How cosigning works in Mimo
When creating a profile on Mimo, you'll have to submit a `claim` as well as a `signedClaim` (from our designated consigner). If you are the only one to register a profile at the time, then the profile goes to you. If the profile is already registered then the timestamps of both profiles are compared and whoever's timestamp is earliest gets/retains ownership of the profile. In the event of any conflicts due to lack of consensus algo on orbit, you simply have to resubmit your claim again to get the profile.
