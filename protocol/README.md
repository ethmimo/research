# Mimo Protocol

The Mimo is essentially two components: the database and JS library.

The database is a docstore OrbitDB instance, customized to accept signatures and modify data according to user signatures.

A key consideration to take into account when examining the design of the protocol is that we have taken some compromises which definitely makes Mimo less trustless than wanted in practice, **BUT** user always retain the ability to verify data for themselves. Mimo prioritizes convenience for user and developer experience, but if someone is willing to jump through a few hoops they can always check things for themselves.

## Identifiers

### IDs
The db will hold all data for every Mimo profile and index them by `_id`. Each `_id` is a hash of the name and public key that owns the profile.

`_id = hash(name + public_key)`

A user can't register two profiles with the same name, but any user can register a profile with a taken name. So I can't register @mimo with the same keypair but any number of keypairs can register @mimo.

### Human-Readable Names
Any keypair can register an existing name, so to return the right profile we're looking for we use our [ProfileRank algorithm](ProfileRank.md).

We assume that no good third-party actor will want to purposely use a name this is already in use by others. Even if they really want a particular name, the risk of someone mistaking someone else for them should persuade users to pick a name that hasn't been picked yet. A user's main priority with a human readable name is to be found easily, this doesn't need to be through the coolest vanity name you can find. This being the case, if forced to choose between either of the two, a user will choose a name that makes it easiest to find them and not what looks the most aesthetically pleasing.

To counteract malicious actors, we propose an open source algorithm we call ProfileRank to help retrieve whatever profile a person is searching for. Since no central authority can attest to who is the rightful owner of a name, we instead rely on ProfileRank to return a profile best suited to us.

ProfileRank is to be implemented by Dapp developers.

## Updating Data
All changes to a profile are authorized through digital signatures with one's private key. If I want to change something in my profile I sign a piece of data to be added to my profile. The name of the profile I want to edit and the recovered public key are then hashed to generate an `_id` and any changes will be made on the profile with the corresponding `_id`.

1. generate `data` to be signed
2. `sig = sign(private_key, data)`
3. `public_key = recover(data, sig)`
4. `_id = hash(data.name, public_key)`
5. `data._id = _id`
6. `db.put(data)`

We have gone through painstaking measures to ensure that the protocol works with no extra hardware requirements, all that's needed is a keypair that to sign messages.
