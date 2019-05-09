# Delegation in Mimo

In order to make Mimo smoother to use, you can give other keypairs access to write data on your profile. That way you won't have to go through a Metamask popup to sign a message for every change you want to make.

Again, we use the `_id` mechanism to give access write to other public keys without revealing who exactly has access. When adding data to the profile, an `_id` is generated and compared to the existing `_id`. If there's a match between the `_generated_id` and the `profile._id` then the owner signed the change, if not then we move on the `profile.delegates` and check if `profile.delegates.includes(_generated_id) == true`. This evaluates whether the `_generated_id` is included in our profile's list of delegates.
