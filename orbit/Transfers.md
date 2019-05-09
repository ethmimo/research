# Transfering Your Identity in Mimo

Ownership in Mimo is tied to your `_id`, it hashes both the name of the profile and the public key that owns the profile. Whenever someone wants to update their profile, the name and the signed data they provide is used to generate an `_id`, which is how the system knows where to put the changes.

So to transfer a profile over to a new address we simply have to change the existing `_id` for a new one.

`_old_id --> hash(name + new_public_key)`

Using our `tranfer()` function implemented in the db, we can swap the `_id` of our profile for a new one without changing the `registered_on` timestamp, which is important to prove ownership of a name. To be clear, transferring is **NOT** like a new registration. Calling `register()` will add a new `registered_on` timestamp to your profile.
