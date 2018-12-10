# Searching for Profile in Mimo (ProfileRank)

To counteract malicious actors, we propose an open source algorithm we call ProfileRank to help retrieve whatever profile a person is searching for. Since no central authority can attest to who is the rightful owner of a name, we instead rely on ProfileRank to return a profile best suited to us.

Say our usr, Alice, wants to find their friend Bob on the Mimo network. Only problem is, there are thousands of "@bob"s on the network. How do we find the right one? Thankfully, several of Alice's friends already follow Bob. If several of our friends follow a particular Bob, we can deduce that this Bob is the right one.

Given the lack of consensus algorithm in Mimo, we cannot return "the right profile" when querying the network. Instead, we focus on returning the right one **for you**.

We can do this by returning:
- profiles with a high follower count
- profiles that have the same friends (or followers) as you do.
- profiles that are followed by profiles you follow
