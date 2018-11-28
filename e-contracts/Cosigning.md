# Cosigning in Mimo

Author: **Ghilia Weldesselasie** <br>
Date: _04/10/2018_

---

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
        const _keypair = EthCrypto.createIdentity();

        // We create a getter to allow access to our cosigner's pubkey
        this.address = () => _keypair.address;

        // We create a function that cosigns claims with our cosigner's privkey
        this.cosignRegistration = registration => {

          // We prep the claim by adding timestamp
          registration[timestamp] = Date.now();

          // Our cosigner sign the registration with it's private key
          const signedRegistration = EthCrypto.sign(
            _keypair.privateKey,
            EthCrypto.hash.keccak256(JSON.stringify(registration))
          );

          // We return both the original registration (with a timestamp) and the signed claim
          return { registration, signedRegistration };
        }
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

Meaning if we make a claim on a name before anyone else in the world, but because of issues with consensus on pubsub someone else takes ownership of a name before us, if we can prove that we really did claim this name first (with our timestamp) then can take ownership of the name at any time. This is where our impartial cosigner comes in to back up our claim. Alternatively, no one will be able to take our name from us if we're the first ones to claim it.

Let's call this **Proof of Dibs (PoD)** :)

## How cosigning works in Mimo
When creating a profile on Mimo, you'll have to submit a `registration` as well as a `signedRegistration` (from our consigner). If you are the only one to register a profile at the time, then the profile goes to you. If the profile is already registered then the timestamps of both profiles are compared and whoever's timestamp is earliest gets/retains ownership of the profile. In the event of any conflicts due to lack of consensus algorithm in IPFS, you simply have to resubmit your claim again to get the profile.

**Update 08/10/2018:**
After testing out e-contracts at ETHSF, in my implementation of a State Channel using OrbitDB, I can confirm that transforming an orbit db instance into a "contract" does in fact work. A keypair can also be included to cosign claims meaning that OrbitDB really is a good fit for Mimo. There is still more work to do before a suitable off-chain identity solution is on mainnet, but this is a great step forward.

**Update 28/11/2018:**
# Earliest Registrants
After thorough research we have concluded that ensuring that each profile has a unique name is impossible. So instead of going for truly unique names we decided to go with an approach called **earliest registration**. With this, instead of ensuring uniqueness we designated the first registrant of a name as the true owner. So if 1000 people register `@ghiliweld` after me, I will always be designated as the owner of `@ghiliweld`. All that's stored on the orbitdb is the timestamps for when the profile were created, while returning the earliest registrant is done on the client side by whatever Dapp you're using.
