# <http://p21op.io>

An open-source hub for person-to-one-other-person solidity smart contracts.

## What is Person-to-One-Other-Person (p21op) Software?

**p21op** software conotes an Ethereum smart contract that is programmed to allow only two participants. The participants may be either cooperating or antagonistic. Their status towards each other does not matter. What matters is that the contract obeys certain rules. (See below.)

The first participant in the contract is called the `owner` and is responsible for deploying the contract. The second (and only other) participant is called the `player`. Either party may have written the contract. Alternatively, the parties may have worked together to write the contract. Or, in a third scenario, the parties may choose a third-party contract they found on this site. The sharing of **p21op** software is the primary purpose of this website.

## Motivation

The idea of **p21op** software was motivated by the following considerations:

* By enforcing only two participants, it is hoped that the software will be simplified, thereby lessening the software's attack surface.

* In many cases, the two parties will know each other, further lessening the need for blind trust. In a sense, the outside-world trust between the two parties will have been brought into the world of the smart contract.

* In cases where the two parties do not know each other, it is hoped that because much of the code needed to manage large groups will have been removed, this will further lower complications and potential attack surfaces.

* By abstracting commonalities out of **p21op** software, it is hoped that there will emerge standards around building this type of software. Perhaps there will be an emergence of a standard set of contract templates much like those arising out of the ERC20 token standard.

* Through the idea of smart contract 'smart categories', it is hoped that other 'smart categories' may emerge. Isolating classes of different contracts may allow best-practices to more easily emerge.

## The Rules of P21OP Software

To be considered a **p21op** piece of software, a smart contract must exhibit these characteristics:

* The contract must have a single _owner_. The owner:

  * must have been set in the contract's constructor from the _msg.sender_ value,
  * may not be changed subsequently,
  * must be provided with an un-obstructed ability to retreive all **free** ether in the contract;
  * be able to `kill` the contract if it contains no unspent ether.

* The contract must allow only a single additional participant, normally called the _player_. The player:

  * must be allowed, after a previously agreed upon delay, to retreive all `escrowed` ether,
  * may be removed from the contract by the _owner_ if the escrow account is empty,
  * x

* Both parties agree that if one party retreives what would seem to be the other person's ether (either through a purposefully programmed function or the use a previsouly unforseen "feature"), that the **taking** party is entitled to keep the ether, and that the **giving** party has no recourse, legal or otherwise.

## What Can I Do With p21op Software?

There are a number of different scenerios where one might wish to use p21op software. Perhaps you and a friend wish to share something, for example, birthday wishes. Unlike all other examples of birthday wishes, you and your friend want to make the interaction exciting, challenging and fun. Try the birthday contract below. Another scenerio may be where you wish to hire someone who you know, but you don't know them well enough to know if they do good work. Try the consulting contract below. A final example might be that you desire a simple game of **chicken**--put a little spice in your life--try **nameGame**.

## Examples

Note: The following list will grow with time as the movement takes hold. Do you have an idea for a peice of **p21op** software? Send it to us [here](mailto:info@greathill.com). Here are a few examples of **p21op** software:

1. [nameGame](https://github.com/Great-Hill-Corporation/nameGame)
2. [Gift of Friendship](https://github.com/naterush/gift-of-friendship)
3. [RockPaperScissors](https://github.com/HashFairGames/RockPaperScissors)
