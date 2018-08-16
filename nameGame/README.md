## NameGame

This purposefully interactive and low-stakes smart contract is intended to be used in teaching simple Solidity concepts. The goal is to explore two things: (1) proper 
handling of a contract's ether, and (2) understandable and controlled 'social engineering.' That is, a design is sought that will channel each player's in-game actions 
in an understandable and teachable manner.

## Discussion

The game asks each of two to five players to put up a stake (i.e. `entryFee`). At any time during play, any player may call `playGame` to increment the `current` 
pointer. When the game time expires, `current` wins the pot.

The intended use for this contract is in a classroom or instructional setting. A lively, group interaction is desired. As each player makes his/her moves, the current 
pointer is incremented in an unpredictable way. Perhaps more than one `playGame` call will be mined in a single block. Perhaps one player will purposefully increment 
the `current` pointer so that his/her friend does not win. Perhaps repeated quickly-timed calls will win increase one's chances of winning. Perhaps a single, 
perfectly-timed call will win. After the game is over, the class may discuss what happened, and perhaps play again with a different group of students. The game pays 
out and resets after each game, until the owner of the contract calls `destroyContact`. Of course, the game is intended to be played with `testnet` ether.

An excellent discussion of programming with security in mind in here: <a href="https://www.kingoftheether.com/contract-safety-checklist.html">King of Ether 
Checklist</a>.

## Game Play

#### The contract is deployed with three parameters:

* number of players (between 2 and 5, inclusive)
* duration (between 30 and 300 seconds, inclusive)
* per player entry fee

Once deployed the contract enters the `preGame` state.

#### In the `preGame` state:

1. Interaction with all but the `joinGame` and `destroyContract` functions is disabled,
2. Only the `owner` of the contract may call `destroyContract`, which will return each player's ether and kill the contract.
3. For the `joinGame` function:
    * Players must send exactly the `entryFee` to join the game,
    * Players must send a non-empty `name`,
    * The first player to call the function becomes `current`,
    * The `n`th player to call the function sets the `endTime` marker and causes a transition into the `gameTime` state.

#### In the `gameTime` state:

1. The `joinGame` and `destroyContact` functions are disabled,
2. The `playGame` function is enabled:
    * Players must not send any ether to the `playGame` function,
    * With each call, `current` is incremented modulo `nPlayers`,
    * If `now` is greater than `endTime`, the then `current` player wins the pot and the contract is returned to the `preGame` state.

## Final Points

* Obviously, there could be many improvements to this smart contract; however, I am not looking to add features only to make the contract more secure or more 
understandable.
* The contract is intentionally easy to understand and low-stakes, any suggestions should keep it that way.
* While remote play is possible, the contract is intended to be used interactively (i.e. all players present, in-person) for instructional purposes.
* A focus on careful handling of the ether is primary.
* A focus on controlling understandable player behavior is primary.  
* Please focus comments on improving the instructional quality of the contract, as opposed to adding features. Building an easily taught, thought-provoking, 
interactive smart contract is the purpose of this exercise.

## Reviewer's Comments

The following comment was made by <a href="https://github.com/o0ragman0o">o0ragman0o</a> concerning the timing in the game:

    The game can be set to run over a period of 30 seconds to 5 minutes and ends when a duration
    of initial block time + game duration is exceed. Given that miners at not required by the protocol
    to prove an accurate timestamp at the time of mining, there is no guarantee that the subsequent
    timestamps will be linearly or even statistically increasing over such a short period.

    Consider the contract is launched with 30 second duration: Let blk1 be the timestamp as seen by
    last player to enter. Time end == blk1 + 30 seconds. Note that the block containing the last
    player actual TX is blk2 with a timestamp ~12:00:14 but under normal statistical variance may
    well exceed 12:00:30. This means the very first call to playGame on blk2 will ensure a win for player 1.

    Consider the contract is launched with 300 second duration or approximately 21 blocks. There are
    21 chances of a random miner's recording a misconfigured timestamp beyond the contract's duration
    and forcing an early end to the game. This is not a security risk but might present unexpected behaviour.```

<a href=“https://github.com/veox”>veox</a> pointed out three issues related to gas usage:

    1. Gas fees for players are uneven (transactions that clear storage get a gas reimbursement, transactions
       that set to non-zero from zero use more gas). References the yellow paper, page 10, section 9.2).

    2. Using 0 to specify a state machine's state may have unexpected results in Solidity, since that's
       the default for state variables. The contract's logic may need review in light of this fact. In
       our case, this is not a problem.

    3. Uneven gas cost may be seen as an oversight, or, in our case, a feature because we want to point out
       that this is something a solidity programmer needs to consider in the design of his/her dapp.

## Open Issues

**preGame has been renamed--update README.md file**
On suggestion of one reviewer, I change the name of the preGame and gameTime modifiers, but I didn't update the README.md file which still references these names.

**Push payment is used with `send()` and may block the dapp**
```
if (now >= endTime) {
    if (players[current].addr.send(entryFee*nPlayers))
        nPlayers = 0;
}
```

If the send fails (due to receiving account having an expensive fallback), the ether will be stuck with this dapp, `nPlayers` won't get reset, and the dapp will be 
effectively blocked (no new game can be started, only course of action is `selfdestruct` (EDIT: to `owner`, which is not implemented)).

This might not be a big issue for your proposed classroom scenario, where you're likely to be reachable to perform such administrative actions; however, it's still a 
burden.

For an examlple of how bad this can get, see [KotET's checklist](https://www.kingoftheether.com/contract-safety-checklist.html) (section C2) and the detailed [issue 
description](https://www.kingoftheether.com/postmortem.html) linked therein.

Related: #1, #6

Relevant: suggestion at the end of #7.

**Pregame player can prevent contract suicide.**
 ```
   function destroyContract() isOwner() isPreGame() {
        // Only owner may kill the contract, but everyone gets their stake back
        //
        // This function tries to return each player's entry fee. Note: one
        // or more of the refunds may fail, in which case the 'current' gets
        // the remaining stake. Players must correct for this externally.
        for (uint i=0;i<nPlayers;i++)
            if (!players[i].addr.send(entryFee))
                throw;

        suicide(players[current].addr);
    }
```

This function throws if a send fails meaning that `suicide(players[current].addr);` will not be called.

Consider that the owner tries to destroy but a player's contract's default function in not set to payable or not configured at all which defaults to a throw on 
payments:
```
contract forcedThrowOnPayment {
    function ()  { }
}
```

I've thought of this technique to fix this. Remember, my goal is to keep it very simple to explain so I can use it for teaching.

During the joinGame function, the function sends 1 wei to each player who registers. If that send fails, the user is rejected. If it succeeds, the player is accepted. 
In this way I know that the new player  is either not a contract, or at least does not have an 'unsendable' default function.

Then, during contract destruction, the code sends each player (entryFee - 1 wei).


The safest model is the 'withdraw' paradigm where a user is able to manually withdraw funds from a contract.  In your case the teacher could put a 'halt' during 
pregame and it's up to the players to reclaim their own funds.  So a withdraw function might look like:

```
bool runState = true;

function halt() isOwner {
    runState = false;
}

function withdraw()  {
    if (!runState && findPlayer())  {
        players[playerIndex()].addr = "0x0"  //force address mismatch to prevent multiple claims
        nPlayers--
        msg.sender.send( entryFee ); // not throwing on send means player can't prevent suicide. Owner gets failed funds.
        if(0 == nPlayers) suicide(owner);
}

function playerIndex() internal returns (int) {
        for (uint i=0;i<nPlayers;i++) {
            if (addr == players[i].addr) {
                return i;
            }
        }
```


Doesn't this also have the same issue if one or more players never reclaim their ether? That is, if 0 == nPlayers never hits. And, just to be clear, if the receiving 
address recursively calls back in, findPlayer would fail and it wouldn't re-enter the if statement in withdraw?

One very nice thing to teach might be the withdraw function without the 'findPlayer' call which would be re-enterable and be a demonstration of the DAO hack. And then 
fix it to demonstrate.

I think I will also replace the findPlayer with simply using playerIndex() just to eliminate some code. Less code == easier to teach.  I'll leave this open until I 
implement it.



Your right, a non-claiming player could prevent contract suicide.

'just to be clear', I see my code is also bugged... as written `playerIndex()` can return 0 as a valid index but will also return 0 as a failed search meaning a 
non-player address can claim player 1's fee.

I've found in my own contracts that it's a good idea to protect index zero for just this kind of reason, i.e does `0` mean a number, a failure, or uninitialised data?

This can be worked around simply by beginning your indexing from 1 rather than 0.

```
function playerIndex(address addr) internal returns (uint) {
    for (uint i=1; i <= nPlayers; i++) {
        if (addr == players[i].addr) {
            return i;
        }
    return 0;
    }
}

function withdraw()  {
    uint player = playerIndex(msg.sender);
    if (runState || (player == 0)) throw;
    delete players[player];  // delete player store to prevent reclaiming. Also refunds gas.
    nPlayers--;
    msg.sender.send( entryFee ); // not throwing on send means player can't prevent suicide. Owner gets failed funds.
    if(0 == nPlayers) suicide(owner);
}

```


> ```
> delete players[player];  // delete player store to prevent reclaiming. Also refunds gas.
> nPlayers--;
> ```

Does the ‘delete’ pull the remaining items in the array down one index? What if player number ‘2’ withdraws. Won’t the playerIndex function spin 
through index ‘2’ and hit a deleted item?

I know you’re just throwing out ideas, but I want to make sure I understand.

Can I ‘delete’ an item in an array, and then later index it as I would if player ‘2’ withdrew first, and then player ‘3’ came in later?

Plus, wouldn’t player ‘4’ drop off the end of the list if player ‘2’ was deleted and the nPlayers simply decremented?

By the way—remember—I’m doing this to later use it as a teaching aide, so I’m just documenting the conversation. I think my students may be interested 
in the types of things one must think about that don’t exist when one is not programming money.

# Thanks.

Thomas Jay Rush
Great Hill Corporation
jrush@greathill.com
(610) 519-9413

> On Nov 15, 2016, at 1:23 AM, o0ragman0o notifications@github.com wrote:
> 
> Your right, a non-claiming player could prevent contract suicide.
> 
> 'just to be clear', I see my code is also bugged... as written playerIndex() can return 0 as a valid index but will also return 0 as a failed search meaning a 
non-player address can claim player 1's fee.
> 
> I've found in my own contracts that it's a good idea to protect index zero for just this kind of reason, i.e does 0 mean a number, a failure, or uninitialised data?
> 
> This can be worked around simply by beginning your indexing from 1 rather than 0.
> 
> function playerIndex(address addr) internal returns (uint) {
>     for (uint i=1; i <= nPlayers; i++) {
>         if (addr == players[i].addr) {
>             return i;
>         }
>     return 0;
>     }
> }
> 
> function withdraw()  {
>     uint player = playerIndex(msg.sender);
>     if (runState || (player == 0)) throw;
>     delete players[player];  // delete player store to prevent reclaiming. Also refunds gas.
>     nPlayers--;
>     msg.sender.send( entryFee ); // not throwing on send means player can't prevent suicide. Owner gets failed funds.
>     if(0 == nPlayers) suicide(owner);
> }
> 
> —
> You are receiving this because you modified the open/close state.
> Reply to this email directly, view it on GitHub, or mute the thread.


No delete doesn't change the indexing and you can still write back to the deleted index with new data without effecting the rest.
There's also some gas efficiency calculations that can be gained by rewriting into array elements rather  than first write.

