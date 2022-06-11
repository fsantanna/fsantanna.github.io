# Freechains

<img src="../twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna)

<img src="tragedy.jpeg" align="right" width="250" alt="Image from Financial Times">

<!--
https://www.ft.com/content/ec74ce54-d3e1-11e7-8c9a-d9c0a5c8d5c9
-->

[Freechains][1] is a permissionless social media protocol with integrated
reputation.

A user posts a message to a chain (a forum) and all other users subscribed to
the same chain eventually receive the message.
Users spend their reputation to post new messages and gain reputation from
consolidated posts.
Users can like and dislike messages from other users, which transfer reputation
between them.

# Freechains vs Tragedy of the Commons

Social media platforms suffer from the [tragedy of the commons][2]:
    the more users are in the platform,
    the more it is used,
    the more it is abused,
    the more it decays.
Users act on their own interests, contrarily to the common good, causing the
platforms to collapse.
There is no principled and localized coordination, but only centralized and
generic counter-abuse measures.

Freechains relies on a reputation system that embraces locality and
subjectivity to escape the tragedy of the commons:

1. Users can create forum chains with their own subjective rules (netiquettes).
2. Creators receive initial reputation tokens (`reps`) to distribute to new users.
3. Users need `reps` to post.
4. Consolidated posts generate new `reps` to their authors.
5. Likes and dislikes transfer `reps` between users.

These rules
    support local communities with common interests (rules `1` and `2`),
    prevent abusive behavior from free riders (rule `2`),
    encourage authoring and steady growth (rule `4`), and
    lead to descentralization of power (rules `4` and `5`).

# So, yet another blockchain?

Freechains requires a consensus mechanism to prevent free riders to double
spend `reps` to abuse the system.

Bitcoin was the first protocol to ensure consensus on a permissionless newtork.
However, Bitcoin and cryptocurrencies in general are not suitable for social
media:

- They enforce a unique timeline to preserve value and immunity to attacks.
- They lean towards concentration of power due to scaling effects.
- They impose an extrinsic economic cost to use the protocol (mining or fees).

In particular, a unique timeline implies that all Internet content should be
subject to the same consensus rules, which neglects all subjectivity that is
inherent to social media.

Freechains, on the other hand, relies on the scarcity of posts to reach
consensus (*proof-of-authoring*), which is based on human work and is localized
inside each chain.

Technically, Freechains is not a linked list blockchain, but a causal graph
of posts (a Merkle DAG) with an overlay consensus list, which may itself affect
branches of the DAG if they have inconsistencies (e.g., double spend).

We are currently working on a [paper][3] that describes the consensus and
reputation mechanism of Freechains.

[1]: https://github.com/Freechains/README/
[2]: https://en.wikipedia.org/wiki/Tragedy_of_the_commons
[3]: http://ceu-lang.org/chico/papers/fc_xxx22_pre.pdf

---

- Do cryptocurrencies lead to centralization of power?
- Are extrinsic costs acceptable for social media interactions?
- Is *proof-of-authoring* reasonable as a scarce resource towards consensus?

Comment on <img src="../twitter.png" style="vertical-align:middle">
[@\_fsantanna](https://twitter.com/_fsantanna/status/TODO).
