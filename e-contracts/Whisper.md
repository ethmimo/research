# Whisper + E-Contracts

Using Whisper in e-contracts to solve the consensus issues holds some promise. If instead of using a [cosigner approach](Cosigner.md), we can have our e-contract subscribe to certain topics on the whisper network and modify it's data according to messages it catches.

In the context of identity, a user that wants to register a name would simply have to submit a message on the whisper network. Whenever a registration is picked up by our contract it would give ownership to whoever is claiming the name.

In the event that our contract picks up a registration made after us before ours, whenever our registration reaches the contract ownership would simply go to us, if it has an earlier timestamp.

Using whisper might also be the key to solving double spending issues in e-contracts.
