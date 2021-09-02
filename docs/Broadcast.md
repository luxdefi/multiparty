---
title: Broadcast
author:
  - J.-P. Aumasson
  - A. Hamelink
  - L. Meier
date: 14-07-2021
---

## Reliable Broadcast

Let $P^{(1)}, \ldots, P^{(n)}$ be a set of $n$ participants in a protocol, and let $t$ be the maximum number of participants that may be malicious at any given epoch.

For a general threshold signature scheme, the integer $t$ may take any value in $\{0,1,\ldots, n-1\}$, since $t+1$ corrupted parties may reconstruct the full signing key by combining their individual shares. The signing protocol itself can then be run by any subset $S\subseteq \{P^{(1)}, \ldots, P^{(n)} \}$, as long as $|S| > t$. In the worst case scenario, $S$ may contain only a single honest participant, and we must therefore assume a _dishonest majority_ in $S$.

In this setting, it may be impossible to guarantee that the protocol acheives either _guaranteed output delivery (the adversary cannot prevent honest parties from receiving the final output)_ or _fairness (either all honest parties receive the protocol output, or none do)_. Most often, this is due to the protocol assuming the existance of a _reliable broadcast_ channel, which guarantees that all participants receive the same message, even when the broadcaster may be malicous. When implementing a protocol, we must often rely only on a _point-to-point_ network between participants, and naively broadcasting a message by sending it to each party individually may fail when a corrupt party decides to send different messages to different parties. In fact, a reliable broadcast primitive is equivalent to a consensus primitive, since it requires all participants to agree on the message which was broadcast. Given that consensus is impossible to acheive when $t < n$, we must look at other ways to acheive a broadcast primitive.

## Broadcast with abort

When a dishonest majority is assumed, it is possible to implement a broadcast primitive which guarantees that either all honest participants agree on the sent message, or otherwise abort. This channel is due Goldwasser and Lindell and we will refer to it as an _"echo broadcast"_. Unfortuanately, it allows an adversary to force an abort by simply sending two different messages during a broadcast round.

Suppose each participant $P^{(i)}$ is instructed in round $k$ to reliably broadcast some message $x^{(i)}$ to all other parties. The broadcast is done as follows, from the perspective of $P^{(1)}$:

- $P^{(1)}$ sends $x^{(1)}_j = x^{(1)}$ to each $P^{(j)}$, and stores $x^{(1)}_1 \gets x^{(1)}$.
- Upon reception of $x^{(j)}_1$ from $P^{(j)}$, store $x^{(j)}_1$.
- Compute $V^{(1)} \gets \mathsf{H}(x^{(1)}_1, \ldots, x^{(n)}_1)$ and deliver $(x^{(1)}_1, \ldots, x^{(n)}_1)$ to round $k+1$.
- When instructed by round $k+1$ to send message $y^{(1)}_j$ to $P^{(j)}$, send $(y^{(1)}_j, V^{(1)})$ instead.
- Upon reception of $(y^{(j)}_1, V^{(j)})$ from $P^{(j)}$, abort if $V^{(j)} \neq V^{(1)}$, otherwise deliver $y^{(j)}_1$ normaly to round $k+2$.

<!-- ## Broadcast with identifable abort
 -->

<!-- In order to attribute fault in the situation where $V^{(j)} \neq V^{(1)}$, we need a mechanism to detect whether a party has sent two different messages $x^{(j)}_1 \neq x^{(j)}_2$.

In this case, we instruct the participants to send (without reliability) the full set of messages $(x^{(1)}_1, \ldots, x^{(n)}_1)$ to all, so that each party can check whether two different messages were sent. Messages must therefore be signed with the sender's public key (independent from any key material generated by the protocol), and the receiver must verify the signature upon reception. Additionally, the signed message must be prefixed by a some session identifier which is unique to each protocol execution, as to prevent a participant from resending a valid message originating from a previous execution. This session ID cannot be generated by the protocol, since it is requires agreement among the participants, i.e. consensus.

One way of obtaining a unique session ID is by simply using a counter which is incremented before each protocol execution (even failing ones). Unfortunately, this requires the participants to maintain additional state which may not always be practical.

Another solution is to use a public randomness source, for example usign the DRAND network. -->

<!-- cite lindell  -->

<!-- ## Broadcast considerations

Abort identification requires the use of a reliable broadcast channel.
Unfortunately, this condition is hard to meet in practice when the network is modeled as point-to-point.
The protocol generally requires the message from the first round to be reliably broadcast, and so for this step we use the _"echo broadcast"_ from Goldwasser and Lindell.

In the keygen and refresh protocols, the broadcasted message is a commitment, so we need to add an extra round of communication to make sure the parties agree on the first set of messages.
For the signing protocol, the "echo round" can be done in parallel during the second round.

The main disadvantage of this approach is that an adversary can cause an abort by simply sending different messages to different parties.
It therefore makes little sense to implement the indentifiable abort aspect of the signing protocol since an adversary could simply cause an anonymous abort at the start of the protocol.

On the flip side, the user of the library only needs to provide authenticity and integrity between the point-to-point connections between parties. -->