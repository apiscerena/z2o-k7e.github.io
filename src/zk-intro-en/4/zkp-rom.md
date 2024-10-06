# Random "Challenges"

*Exploring Zero-Knowledge Proofs Series (Part 4)*

> “Challenges are at times an indication of the Lord's trust in you.”  
> — D. Todd Christofferson

This article continues our lengthy discussion on the mechanisms and principles behind zero-knowledge proofs, aiming to help readers understand the general framework of this "modern cryptographic tool." The article has a few mathematical formulas.

## Interactions and Challenges

The zero-knowledge proof systems we introduced earlier are all interactive, requiring the verifier Bob to provide one or more random numbers during the interaction to challenge the prover. For instance, in the "map 3-coloring problem" (see *Part 2*), the verifier Bob needs to repeatedly select a random edge to challenge Alice's solution until he is satisfied, while Alice’s chances of cheating decrease exponentially. Bob’s trust in the proof fundamentally depends on whether the random numbers he selects are truly random. If Alice can predict Bob’s random numbers in advance, disaster ensues—the real world degenerates into the "ideal world", and Alice effectively becomes a "simulator", using her superpowers to fool Bob.

In *Part 3*, we analyzed the Schnorr protocol. Although the verifier Bob only needs to select a single random number `c` to challenge Alice and have her compute a value `z`, Bob absolutely cannot allow Alice to gain any knowledge about `c` beforehand. Otherwise, Alice could also become a simulator.

The importance of random numbers cannot be overstated:

> Random challenges form the **foundation of trust** in interactive zero-knowledge proofs.

However, the **interactive process** limits the application scenarios. What if we could turn an interactive zero-knowledge proof into a **non-interactive** one? That would be incredibly exciting. Non-interactive means the proof process consists of just **one round** — Alice directly sends a proof to Bob for verification.

Non-interactive Zero Knowledge, abbreviated as NIZK, mean that the entire proof is encoded into a **string**. This string can be written on a piece of paper and sent to any verifier via email, chat tools, or other means. It can even be uploaded to GitHub, where anyone can download and verify it at any time.

In the blockchain world, **NIZK** could be part of the consensus protocol because a transaction **needs to be verified by multiple miners**. Imagine if the sender of the transaction had to interact with each miner, allowing them to challenge the transaction. The consensus process would be incredibly slow. But with non-interactive zero-knowledge proofs, the transaction could **simply be broadcast to all miners** for them to verify on their own.

Some may ask: Why not just let one miner challenge the transaction? Encode the interaction script between the miner and the sender into a proof, then broadcast it to other miners. Wouldn’t they trust this challenge process? But clearly, this would require trusting the first miner as a third party. A third party? That doesn’t sound like a good idea…

NIZK, however, seems ideal—no third party to profit from the interaction.

## The Confusion Brought by "Non-Interactivity"

If NIZK exists, it would be far more powerful than interactive proofs.

+ Interactive proofs can only establish trust with a single verifier, while NIZK can establish trust with multiple verifiers, or even with everyone.
+ Interactive proofs are only valid at the time of interaction, while NIZK remains valid indefinitely.

> NIZK can transcend both **space** and **time**.

Sounds amazing, doesn’t it? But…

Let’s revisit a key conclusion from the previous section:

> Random challenges form the **foundation of trust** in interactive zero-knowledge proofs.

But what happens when NIZK loses the challenge process?

We’ve previously discussed how to prove the zero-knowledge property (see *Part 2*), where a simulator (an algorithm) interacts with the verifier (Bob) in the ideal world, and the verifier Bob cannot distinguish whether he is interacting with the real Alice or a simulator.

Now, consider the **non-interactive** nature of NIZK. Suppose "I" present "you" with a piece of paper containing a "real" proof `X`, and you believe me after reading it. Since the protocol is zero-knowledge, if I were replaced by a simulator, the simulator could "forge" a fake proof `Y` that would also convince you.

Here’s the problem:

+ How do you distinguish between `X` and `Y`? Which one is real? Of course, you can’t distinguish between them because the protocol is zero-knowledge — you must not be able to distinguish.
+ I can just as easily show you `Y`. Wouldn’t that mean I can deceive you?

Doesn’t this seem problematic? Take a moment to think about this for two minutes.


(Two minutes later…)


Since NIZK lacks interaction, it also lacks the challenge process. Alice is fully responsible for calculating every bit of the proof, and theoretically, Alice can write whatever she wants—there’s no stopping her. For instance, Alice could write a fake proof `Y` from the "ideal world".

Those who deeply understand the simulator concept may recognize a crucial point here: The simulator must only construct `Y` in the ideal world. In other words, `Y`, being such an evil thing, can only exist in the ideal world and cannot enter the real world to cause harm.

Keep thinking…

There’s an even deeper issue. Recall the "map 3-coloring problem". The reason the simulator cannot wreak havoc in the "real world" is that it has the **superpower of time-rewinding** in the ideal world, but no such black magic exists in the real world. The **non-existence of such powers** in the real world is the key.

Moreover, in NIZK, there is **no interaction**, leading to a severe consequence: The simulator cannot use the **superpower of time-rewinding** and thus cannot differentiate the prover’s behavior between the two worlds.

In other words, when faced with any NIZK system, it seems the "simulator" can no longer be a lofty entity. It seems to fall down to earth, becoming just an ordinary mortal. If, I say if, we assume the simulator no longer has superpowers, it means Alice and the simulator are indistinguishable. Alice can become a simulator. Following this logic, Alice could deceive Bob at will in the real world, making the proof system lose its soundness. Conclusion: No NIZK is sound.

Something must be wrong…

In the analysis above, we mentioned the absence of interactive challenges. Indeed, if Bob does not participate in Alice’s proof generation, and every bit of the proof is provided by Alice, the proof itself seems to lack any "foundation of trust" for Bob. This doesn’t seem to make sense intuitively.

Does this mean that without Bob’s participation, there’s **no way** to establish a "foundation of trust"? Where else can we find this foundation of trust?

The answer is: **A third party**!

Wait… Aren’t protocols supposed to involve just two parties, Alice and Bob? Where does this third party come from?

We need to introduce a third party in a special way, and there’s more than one method. Let’s first explore the first method.

(Tearfully: Didn’t we agree not to introduce a third party?)

## Revisiting the Schnorr Protocol

Let’s revisit our old friend — the **Schnorr protocol**, a three-step protocol. In the first step, Alice sends a commitment; in the second step, Bob sends a random challenge number, and in the third step, Alice responds to the challenge.

![](img/schnorr.png)

Let’s see how we can turn this three-step Schnorr protocol into a one-step protocol.

In the second step of the Schnorr protocol, Bob needs to provide a random challenge number `c`. We can have Alice calculate this challenge number using the following formula, thereby eliminating the need for the second step:

```c = Hash(PK, R)```

Where `R` is the elliptic curve point Alice sends to Bob, and `PK` is the public key. Take a good look at how the Hash function calculates `c`. This formula achieves two goals:

1. Alice cannot predict `c` before generating the commitment `R`, even though `c` is ultimately calculated by Alice.
2. `c`, calculated through the Hash function, is uniformly distributed in a finite field and can be considered a random number. (*Note: Let’s temporarily accept this, and we’ll discuss it in more detail later.*)

Note: Alice must not be able to predict `c` before generating `R`. Otherwise, Alice would effectively gain the superpower of time-rewinding, allowing her to fool Bob at will.

A cryptographically secure Hash function is **one-way**. Examples include SHA256, SHA3, and blake2. So, although Alice calculates `c`, she has no ability to cheat by choosing `c`. Once Alice generates `R`, `c` is essentially fixed. We assume that Alice, a mere mortal in the real world, cannot reverse-engineer the Hash function.

![schnorr-nizk](img/schnorr-nizk.png)

In the diagram above, we use the Hash function to merge the three-step Schnorr protocol into a one-step process. Alice can directly send: `(R, c, z)`. Since Bob has the public key `PK`, he can calculate `c` himself. Thus, Alice only needs to send `(R, z)`.

With slight modifications, we get a **digital signature** scheme. A digital signature is essentially Alice showing Bob a string, say, "The sun sets behind the mountains; the Yellow River flows into the sea", and proving that this poem was presented by Alice by signing something. This signature proves her identity and links it to the poem.


## Viewing Digital Signatures from a NIZK Perspective

Loosely speaking, a digital signature scheme proves two things: (1) I possess the private key, and (2) the private key is linked to the message.

First, I need to prove my identity, which is straightforward — this is exactly what the Schnorr protocol does. It can prove the statement "I possess the private key" to the other party. This proof process is zero-knowledge: it does not leak any information about the private key.

Now, how do we link this to the poem? We modify the calculation of `c` as follows:

```m = "The sun sets behind the mountains; the Yellow River flows into the sea"c = Hash(m, R)```

Here, to ensure that attackers cannot easily forge signatures, we rely on the discrete logarithm problem (DLP) and the assumption that the Hash function satisfies second preimage resistance.

*Note: To strictly ensure the unforgeability of the digital signature, we need to prove that the Schnorr protocol satisfies a stronger property called **Simulation Soundness**. For more details, refer to [2].*

![](img/schnorr-sig.png)

The diagram above shows the well-known **Schnorr signature scheme** [1]. There’s also an optimization here: instead of sending `(R, z)` to Bob, Alice sends `(c, z)` because `R` can be calculated from `c` and `z`.

*Note: Why is this an optimization? Current attacks on elliptic curves include Shanks’ algorithm, the Lambda algorithm, and Pollard’s rho algorithm. Remember, their algorithmic complexity is about $O(\sqrt{n})$ [3], where `n` is the bit length of the finite field. If we use a finite field close to `2^256`, meaning `z` is 256 bits, then the elliptic curve group size would be close to 256 bits. Taking the square root of `2^256` gives `2^128`, so a 256-bit elliptic curve group offers 128-bit security. Therefore, a challenge number `c` only needs to be 128 bits. This means that sending `c` is more space-efficient than sending `R`, which requires at least 256 bits. The combined size of `c` and `z` is 384 bits, about `1/4` smaller than the popular ECDSA signature scheme. The Bitcoin development team is preparing to replace ECDSA with a Schnorr-like signature scheme — muSig [4], which offers more flexible support for multi-signatures and aggregation.*

The method of using a Hash function to turn an interactive proof system into a non-interactive one is called the **Fiat-Shamir transformation** [5], proposed in 1986 by cryptography pioneers Amos Fiat and Adi Shamir.

## Rebuilding Trust — The Magic of the Random Oracle

Earlier, we mentioned that without challenges, the proof seems to lose its **foundation of trust**. But in the Schnorr signature scheme, the Hash function plays the role of the **challenger**, a role formally known as the **Random Oracle** [6].

But why use a Hash function here? In fact, when Alice needs to generate a public random number, she requires a "random oracle". What is this?

It’s time to get creative!

Imagine a world where there’s a **magic oracle** in the sky. This oracle holds a two-column table: the left column contains strings, and the right column contains numbers. Anyone, including you, Alice, or Bob, can send strings to the oracle.

![](img/ro.png)

When the oracle receives a string, it looks it up in the left column of its table. There are two possible outcomes:
+ Case 1: If the string isn’t found, the oracle generates a **truly random number**, records the string and number in the table, and returns the random number to the mortal on Earth.
+ Case 2: If the string is already in the table, the oracle simply returns the corresponding number from the right column.

You’ll notice that the oracle behaves like a random number generator, but with a twist: it returns the same number when given the same string. This oracle is the legendary **Random Oracle**. In the process of merging the Schnorr protocol, we actually need such a random oracle, not just a Hash function. What’s the difference?

- The random oracle returns a truly random number with a uniform distribution for each new string.
- The Hash function does not generate a truly random number with a uniform distribution.

So why did we use a Hash function earlier? Because in the real world, **a true random oracle does not exist**! Why not? A Hash function cannot generate true random numbers because it is a **deterministic** algorithm — no additional randomness is introduced beyond the input parameters.

However, A cryptographically secure hash function "seems" to act as a "pseudo" random oracle. Thus, the merged security protocol requires an additional strong assumption:

> Assumption: A cryptographically secure Hash function can approximately simulate the legendary "random oracle."

Since this assumption cannot be proven, we must either trust it or treat it as an axiom. Incidentally, the broad collision resistance of Hash functions enables their output to simulate random numbers. In many cases (though not all), attacking a Hash function is very difficult, so many cryptographers boldly use them.

The security model that does not rely on this assumption is called the **"standard model"**, while the security model that uses this assumption is not called the **"non-standard model"**. Instead, it has a fancy name: the **"random oracle model"**.

There are two types of people in the world: those who like sweet tofu pudding and those who don’t. Similarly, there are two types of cryptographers: those who like the random oracle model and those who don’t [6].

## Building the Foundation — The Kidnapped Oracle

After the **Fiat-Shamir transformation**, the Schnorr protocol gains the NIZK property. This is different from the SHVZK property we proved earlier, which requires the verifier to be honest. NIZK, however, no longer imposes any unrealistic requirements on the verifier since there is no interaction, and the question of an honest verifier doesn’t arise.

*Note: What happens if the verifier Bob is dishonest? In that case, Bob might be able to extract Alice’s knowledge. However, whether the three-step Schnorr protocol itself satisfies "zero-knowledge" remains unknown. In *Part 3*, we only proved that it satisfies a weaker property: SHVZK.*

However, when the Schnorr protocol transforms into a non-interactive zero-knowledge proof system, it truly becomes "zero-knowledge". But you might now ask: This argument sounds reasonable, but can it be proven?

It’s time. "Bring in the simulator!"

How do we use the simulator trick to construct an "ideal world"? Think about it: we’ve previously used "time-rewinding" and the ability to modify the "random number conveyor belt" to let the simulator cheat. But now there’s no interaction, which means: the superpower of time-rewinding cannot be used, and Bob’s random number conveyor belt doesn’t exist, so the ability to modify it also cannot be used!

> But the simulator still needs some superpower to build the foundation of trust.

(If the simulator could cheat without any superpower, that would prove the protocol is unsound.)

By now, you may have guessed: the simulator will tamper with the **random oracle**.

Let’s first construct an ideal world to prove "zero-knowledge". In the ideal world, the simulator "kidnaps" the oracle. When Bob asks the oracle for a random number, the oracle doesn’t give a true random number but instead returns a number that Zlice (the simulator pretending to be Alice) has prepared in advance (while maintaining a uniform distribution to ensure indistinguishability). The oracle, helpless, returns a number to Bob that looks random but has a backdoor. **The backdoor is that the number was pre-selected by Zlice**.

![](img/schnorrsig-sim1.png)

+ **Step 1**: Zlice randomly selects `z` and `c`, then calculates `R' = z*G - c*PK`.  

![](img/schnorrsig-sim2.png)


+ **Step 2**: Zlice writes `c` and `(m, R')` into the oracle’s table. 

![](img/schnorrsig-sim3.png)


+ **Step 3**: Zlice sends the signature `(c, z)` to Bob.  

![](img/schnorrsig-sim4.png)


+ **Step 4**: Bob calculates `R = z*G - c*PK` and sends `(m, R)` to the oracle. The oracle returns `c’`. Note that Bob’s calculated `R` and Zlice’s calculated `R'` are equal.  

![](img/schnorrsig-sim5.png)


+ **Step 5**: Bob verifies if `c == c'`. If the random number from the oracle matches the number sent by Zlice, the signature is valid. 

By kidnapping the oracle, Zlice can predict the random number in advance, achieving the same effect as time-rewinding. We have now proven the **existence** of the simulator Zlice, which means we’ve proven NIZK.

Next, let’s prove the **soundness of the protocol**:
1. Imagine another ideal world, where an entity called the **extractor** has also kidnapped the oracle. When innocent Alice asks the oracle for a random number, the oracle returns `c1`. The extractor, peeking at the table, sees `c1`. After Alice calculates `z1`, the extractor uses its time-rewinding superpower to rewind Alice to Step 
2. Alice asks the oracle for another random number. Since Alice sends the same string `(R, m)`. 

As before, the oracle should return `c1`. However, the extractor has tampered with the oracle and changed the table entry for `(R, m)` to a different number `c2`. After Alice calculates `z2`, the extractor completes its task by computing Alice’s private key `sk` using the following equation:

```sk = (z1 - z2)/(c1 - c2)```

## Fiat-Shamir Transformation — From Public-Coin to NIZK

Not only can the **Schnorr protocol** be transformed using the Fiat-Shamir method, but any **Public-Coin Protocol** can be compressed into a single interaction, creating a non-interactive proof system. This transformation technique comes from the paper *"How to Prove Yourself: Practical Solutions to Identification and Signature Problems"* by Amos Fiat and Adi Shamir, published at the Crypto conference in 1986 [5]. Some say the technique originated from Manuel Blum [6].

To reiterate, in a Public-Coin Protocol, the verifier Bob only does one thing: generate a random number and challenge Alice. Through the Fiat-Shamir transformation, each of Bob’s challenges is replaced with a call to a random oracle.

In practice, the random oracle is implemented using a cryptographically secure Hash function (not just any Hash function — you must use a cryptographically secure one). The Hash function’s input should include all previous context. The diagram below is an example to help you quickly understand the Fiat-Shamir transformation process.

![](img/fs-transform.png)
    
Earlier, we mentioned that in non-interactive proof systems, a third party is needed to build the "foundation of trust", allowing Bob to fully trust the proof constructed by Alice. Here, the third party is the **oracle**. In technical jargon, this is called the **Random Oracle**. The oracle isn’t a real third party but a virtual one. It exists in both the real world and the ideal world. In the real world, the oracle is a responsible, quiet gentleman. In the ideal world, it is kidnapped by the simulator.

Public-Coin Protocols have a fancy name: **Arthur-Merlin Games**

![](img/fs-transform.png)

In the image above, the figure in white is Merlin (the wizard), and the handsome guy with the sword is King Arthur. These characters come from the medieval European legend of King Arthur and the Knights of the Round Table.

Arthur is an impatient king who always carries a coin. Merlin is a magical wizard with unlimited computational power. Merlin wants to convince Arthur that a certain statement is true. They engage in a conversation, but since Arthur is lazy, he only flips a coin each time to "challenge" Merlin, and Merlin must respond promptly. After `k` rounds, Arthur should be convinced of Merlin’s statement. Since Merlin has magic, he can always see the result of Arthur’s coin flips [7].

This scenario resembles the **Interactive Proof System** (`IP`) we discussed in *Part 1*, introduced by Goldwasser, Micali, and Rackoff (GMR) in 1985. However, there are differences. In `IP`, both the Prover and Verifier are coin-flipping Turing machines, and the Verifier can secretly flip coins and hide the results from the Prover. In the **Arthur-Merlin Games**, Arthur can only flip coins, and Merlin always knows the outcome of the coin flips.

However, the Fiat-Shamir transformation is only provably secure in the **Random Oracle Model**. Whether the process of using a Hash function to simulate a random oracle is secure is unproven. Moreover, protocols that are secure in the Random Oracle Model might not be secure in reality, as some counterexamples have been found [8]. Worse, in 2003, S. Goldwasser and Y. Tauman demonstrated that the **Fiat-Shamir transformation itself has security counterexamples** [9]. However, this doesn’t mean the Fiat-Shamir transformation can’t be used. It just needs to be applied carefully, without blindly following the process.

Despite these concerns, the Fiat-Shamir transformation is widely used. It’s especially prevalent in general-purpose non-interactive zero-knowledge proofs like **zkSNARKs**. For example, you may be familiar with **Bulletproofs**, and there are lesser-known general-purpose zero-knowledge proof systems like **Hyrax**, **Ligero**, **Supersonic**, and **Libra** (we’ll dive into these in future articles).

## Beware of Security Pitfalls in the Fiat-Shamir Transformation

In the Fiat-Shamir transformation, special attention must be paid to the parameters fed into the Hash function. There have been cases where some parameters were accidentally omitted in the actual code.

For example, in `A, Hash(A), B, Hash(B)`, the second Hash function is missing parameter `A`. The correct approach should be `A, Hash(A), B, Hash(A, B)`. Such mistakes can lead to severe security vulnerabilities. For instance, in Switzerland’s electronic voting system SwissPost-Scytl, several parameters were missing from the Fiat-Shamir transformation in the implementation code. This allowed attackers to not only invalidate votes but also forge votes at will, enabling election fraud [10]. Hence, be extra cautious during implementation. 

Careful readers might review the Schnorr signature and notice that the Hash function seems to omit the parameter `PK`. This is called "Weak Fiat-Shamir Transformation" [11]. However, this particular case doesn’t pose a security problem [3], so don’t try this unless you know what you’re doing.

Recently, some researchers have started working on proving the security of the Fiat-Shamir transformation in the standard model. So far, the progress has been limited, requiring either additional strong assumptions or specific protocol proofs.

## The Power of Interaction
In 1985, after GMR’s paper was repeatedly rejected, it was finally accepted by STOC '85. At the same time, a similar work was also accepted by STOC '85: the paper *"Arthur-Merlin Games: A Randomized Proof System, and a Hierarchy of Complexity Classes"* by László Babai and Shlomo Moran [7], which introduced the **Public-Coin Interactive Protocol** (as the name implies, the Verifier only flips public coins).

King Arthur’s method is simple: repeatedly issue **random challenges** to test Merlin’s statement. This aligns with the intuition we discussed earlier: using random challenges to build the foundation of trust. In the paper, Babai proved an interesting result: `AM[k] = AM[2]`, where `k` represents the number of interactions. It turns out that multiple interactions are equivalent to just two interactions. Two interactions mean Arthur issues one challenge, and Merlin responds.

*Note: There’s another class of problems known as `MA`, where the order of interaction is reversed. In `MA`, Merlin presents the proof first, and then Arthur flips a coin to check it. It has been proven that `MA` can handle a subset of the problems addressed by `AM`.*

Moreover, Babai boldly hypothesized that `AM[poly]` is equivalent to `IP`. This was a fascinating claim: Arthur, a lazy king, could flip coins polynomially many times and successfully challenge Merlin’s claims. And the expressive power of this approach is equivalent to the Interactive Proof System (`IP`) described by GMR. Sure enough, at **STOC '86**, a paper by S. Goldwasser and M. Sipser proved this: `AM[poly] == IP` [12].

![](img/am-ip.png)


This means that repeatedly issuing public **random challenges** is incredibly powerful. It’s equivalent to any interactive proof system. But how does `AM[poly]` relate to other complexity classes? That became the next research focus. 

Three years later, in late November 1989 — exactly 30 years ago — a young cryptographer named Noam Nisan sent an email with his tentative academic conclusions to several cryptographers before heading off on vacation in South America. Little did he know that this email would spark one of the most intense academic races in history. Researchers like M. Blum, S. Kannan, D. Lipton, D. Beaver, J. Feigenbaum, H. Karloff, and C. Lund joined the fray, exchanging ideas and publishing results in a fierce competition. Finally, on December 26 — exactly one month later — Adi Shamir proved the following result:

> `AM[poly] == IP == PSPACE`.
![image-shamir](img/shamir.png)

This result explained the computational complexity of the concept of **valid proof** and revealed the theoretical power of **interactive proof systems**.

*Note: The NP class is a subset of PSPACE. The latter is associated with strategies in games or chess [13].*

László Babai later wrote an article titled *"Email and the Unexpected Power of Interaction"* [14], detailing the thrilling academic competition that unfolded over a month of **email exchanges** and explaining the origins of **interactive proofs**.

## Common Reference Strings — Another "Foundation of Trust"

In addition to the Random Oracle, non-interactive zero-knowledge proof systems can use a **Common Reference String (CRS)** to complete the random challenge. CRS is a random string generated by a trusted third party before the prover Alice constructs the NIZK proof. The CRS must be generated by a trusted third party and shared with both Alice and the verifier Bob.

Yes, you read that right—there’s a **third party** again! Although the third party doesn’t directly participate in the proof, they must ensure the trustworthiness of the random string generation process. This process is known as the **"Trusted Setup"**, a controversial and often disliked but necessary procedure. Clearly, introducing a third party in real-world scenarios can be a headache. What exactly is the CRS used for? How does the Trusted Setup maintain trust? We’ll explore these questions in the next part of this series.


## To Be Continued
The "foundation of trust" in non-interactive zero-knowledge proofs (NIZK) still requires some form of random challenge. One form of challenge is provided by the Random Oracle; another is achieved through a shared random string between Alice and Bob. Both challenge forms effectively introduce a third party, and both must provide a "backdoor" for the simulator to exploit, allowing the simulator to have an advantage in the ideal world — an advantage that must not exist in the real world. 

NIZK is endlessly fascinating, and I am constantly amazed by the brilliant conclusions the pioneers have reached over the past three decades. At the same time, there are still so many unknown corners waiting for the light of inspiration to shine upon them.

This series has received its first pull request on the [GitHub repository](https://github.com/sec-bit/learning-zkp/), courtesy of **Jingyu Hu (colortigerhu)**, who made a minor edit. But in that moment, I felt life. Knowledge exchange and the collision of ideas are truly captivating, aren’t they?

> “Everyone we interact with becomes a part of us.”  
> — **Jodi Aman**

*Acknowledgments: Special thanks to Shengchao Ding, Weiran Liu, and Yu Chen for their professional advice and corrections. Thanks to the teammates from SECBIT Labs (p0n1, even, aphasiayc, Vawheter, yghu, mr) for their editing suggestions.**Acknowledgment: The cryptography research anecdote about Nisan was inspired by Deng Laoshi's article [15].*

### References
- [1] Schnorr, Claus-Peter. "Efficient signature generation by smart cards." Journal of cryptology 4.3 (1991): 161-174.
- [2] Paillier, Pascal, and Damien Vergnaud. "Discrete-log-based signatures may not be equivalent to discrete log." *International Conference on the Theory and Application of Cryptology and Information Security*. Springer, Berlin, Heidelberg, 2005.
- [3] Pointcheval, David, and Jacques Stern. "Security arguments for digital signatures and blind signatures." *Journal of cryptology* 13.3 (2000): 361-396.
- [4] Maxwell, Gregory, Andrew Poelstra, Yannick Seurin, and Pieter Wuille. "Simple schnorr multi-signatures with applications to bitcoin." *Designs, Codes and Cryptography* 87, no. 9 (2019): 2139-2164.
- [5] Fiat, Amos, and Adi Shamir. "How to prove yourself: Practical solutions to identification and signature problems." Conference on the Theory and Application of Cryptographic Techniques. Springer, Berlin, Heidelberg, 1986.
- [6] Bellare, Mihir, and Phillip Rogaway. "Random Oracles Are Practical: a Paradigm for Designing Efficient Protocols." *Proc. of the 1st CCS* (1995): 62-73.
- [7] László Babai, and Shlomo Moran. "Arthur-Merlin games: a randomized proof system, and a hierarchy of complexity classes." Journal of Computer and System Sciences 36.2 (1988): 254-276.
- [8] Canetti, Ran, Oded Goldreich, and Shai Halevi. "The random oracle methodology, revisited." Journal of the ACM (JACM) 51.4 (2004): 557-594.
- [9] Shafi Goldwasser, and Yael Tauman. "On the (in) security of the Fiat-Shamir paradigm." *44th Annual IEEE Symposium on Foundations of Computer Science, 2003. Proceedings.* IEEE, 2003.
- [10] Lewis, Sarah Jamie, Olivier Pereira, and Vanessa Teague. "Addendum to how not to prove your election outcome: The use of nonadaptive zero knowledge proofs in the Scytl-SwissPost Internet voting system, and its implications for cast-as-intended verification." *Univ. Melbourne, Parkville, Australia* (2019).
- [11] Bernhard, David, Olivier Pereira, and Bogdan Warinschi. "How not to prove yourself: Pitfalls of the fiat-shamir heuristic and applications to helios." *International Conference on the Theory and Application of Cryptology and Information Security*. Springer, Berlin, Heidelberg, 2012.
- [12] Goldwasser, Shafi, and Michael Sipser. "Private coins versus public coins in interactive proof systems." *Proceedings of the eighteenth annual ACM symposium on Theory of computing*. ACM, 1986.
- [13] Papadimitriou, Christos H. "Games against nature." *Journal of Computer and System Sciences* 31.2 (1985): 288-301.
- [14] Babai, László. "E-mail and the unexpected power of interaction." *Proceedings Fifth Annual Structure in Complexity Theory Conference*. IEEE, 1990.
- [15] Yi Deng. "Zero-Knowledge Proof: A Slightly Serious Popular Science Article." https://zhuanlan.zhihu.com/p/29491567