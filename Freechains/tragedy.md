# Freechains

<img src="../twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna)

<img src="tragedy.jpeg" align="right" width="250" title="(from Financial Times)">

<!--
https://www.ft.com/content/ec74ce54-d3e1-11e7-8c9a-d9c0a5c8d5c9
-->

[Freechains][1] is a permissionless social media protocol with integrated
reputation.

A user posts a message to a forum (a *chain*) and other users subscribed to the
same forum eventually receive the message.
Users spend reputation tokens, known as `reps`, to post new messages and gain
`reps` as they consolidate.
Users can like and dislike messages from other users, which transfer `reps`
between them.

# Freechains vs the Tragedy of the Commons

Social media platforms suffer from the [tragedy of the commons][2]:
    the more users are in the platform,
    the more it is used,
    the more it is abused,
    the more it decays.
There is no principled moderation, no localized coordination, but only
centralized and generic counter-abuse protections.
Users act on their own interests, contrarily to the common good, leading to the
collapse of platforms.

Freechains, on the other hand, relies on a reputation system that embraces
subjectivity and locality to escape the tragedy of the commons:

1. Users can create forums with their own subjective rules (netiquettes).
2. Creators receive initial `reps` to redistribute to new users.
3. Users must have `reps` to post.
4. Consolidated posts generate new `reps` to authors.
5. Likes and dislikes transfer `reps` between users.

This set of rules
    supports local communities sharing the same interests (rules `1` and `2`),
    prevents abusive behavior from free riders (rule `3`),
    encourages authoring and steady growth (rule `4`), and
    leads to descentralization of power (rules `4` and `5`).

# So, yet another blockchain?

As a matter of fact, Freechains requires a consensus mechanism to prevent free
riders to double spend `reps` to abuse the system.

In this context, Bitcoin was the first protocol to support permissionless
consensus.
However, Bitcoin and cryptocurrencies in general are not suitable for social
media:

- They enforce a unique timeline to preserve value and immunity to attacks.
- They lean towards concentration of power due to scaling effects.
- They impose extrinsic economic costs to use the protocol (mining or fees).

In particular, a unique timeline implies that all Internet content should be
subject to the same consensus rules, which neglects all subjectivity that is
inherent to social media.

Freechains, on the other hand, relies on the scarcity of posts to reach
consensus (*proof-of-authoring*), which is based on human creativity and is
restricted to each forum.

Technically, Freechains is not a linked list blockchain, but a causal graph
of posts (a Merkle DAG) with an overlay consensus list that can itself reject
branches of the DAG if they have inconsistencies (e.g., double spend).

We are currently working on a [paper][3] that describes the consensus and
reputation mechanism of Freechains.

[1]: https://github.com/Freechains/README/
[2]: https://en.wikipedia.org/wiki/Tragedy_of_the_commons
[3]: http://ceu-lang.org/chico/papers/fc_xxx22_pre.pdf

---

- Do cryptocurrencies lead to centralization of power?
- Are extrinsic costs acceptable for social media interactions?
- Is *proof-of-authoring* reasonable as a scarce resource for consensus?

Comment on <img src="../twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna/status/1535752374744227843).
