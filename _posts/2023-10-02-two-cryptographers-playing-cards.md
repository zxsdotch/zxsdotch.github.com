---
title: Two cryptographers playing cards
categories:
  - cryptography
---
I [tooted](https://infosec.exchange/@alok/111161449025589921) the following puzzle:
> Do you teach cryptography? 
> If yes, ask your students to design a protocol for two people to play a cards game over the internet (eg Uno, Gin Rummy, Go fish or whatever). The protocol should be trustless so the players don’t have to rely on a centralized server to deal cards and players shouldn’t be able to peek at the deck unless the game rule allows it.
>
> Give extra points to students who come up with simpler protocols, formal proofs, or an actual implementation.

As I was preparing my writeup describing a solution to this puzzle, I discovered that this puzzle is well known under the name "mental poker". Rivest, Shamir and Adleman [wrote a paper](https://apps.dtic.mil/dtic/tr/fulltext/u2/a066331.pdf) in 1979.

More recently, Nicolas Mohnblatt published an excellent post titled [Mental Poker in the Age of SNARKs](https://geometry.xyz/notebook/mental-poker-in-the-age-of-snarks-part-1). Instead of boring you with yet another protocol, go read Nicolas' work and experiment with their [library](https://github.com/geometryresearch/mental-poker), written using [arkworks](https://github.com/arkworks-rs).
