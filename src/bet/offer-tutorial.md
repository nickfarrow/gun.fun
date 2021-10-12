# Offer tutorial

This is a tutorial that focuses on how to make an offer.
Accounts like [@GoUpNumber] consistently make proposals and look for the best offers.

To see how to make proposals yourself see [`gun bet propose`](./propose.md).

## Who is tutorial this for?

People who want to try out betting with `gun` but don't want to RTFM too much.
You should meet the following criteria:

1. Comfortable with command line applications.
2. Don't mind losing coins due to bad bets or technical malfunctions. You are happy to consider it a donation to science.
3. Happy to report bugs on GitHub.
  
## Install

Follow instructions in [install](../install.md).
Remember to back up your seed words in `$HOME/.gun/seed.txt`.

## Put some coins in your wallet

> 💡 It's a good idea to coin join your coins before sending them to gun (or doing anything really!).
> If you don't feel comfortable betting coins at this stage you can join the fun by using [`gun bet reply`](./reply.md) to post an encrypted message that looks like an offer.

To get a wallet address run:

```
gun address new
```

And you'll see an address.
Send a small amount of coins to it if you're just getting started.

To check when the coins come in you can do `gun -s balance` or `gun -s utxo list` or `gun -s tx list`.
The most important thing is adding the `-s` flag so `gun` checks the blockchain before displaying the result.
Without `-s` the command will just display what's already in the database.

## Find a Proposal

A proposal contains the event that will be bet on and the amount the proposer is willing to risk on it.
To promote this experiment [@GoUpNumber] is frequently posting proposals on twitter and on their Discord channel ([invite link](https://discord.gg/MB27cDJyrR)).

A proposal looks like this:

```
0.005#h00.ooo#/s/EPL/match/2021-10-16/LEI_MUN.vs=MUN_win#ৠöഓཀসƵ୯รແྋŸǢঊಹహĈၐ೨ø໕Ǜœબ೮ǇЧ༃ཆɁঢӛႱ၃ಢͱธྈͰଢƿɧƶവϳॻŹဪ໖ƸØ
```

It contains several parts split up by the `#` character. 

1. `0.005`:  The amount in BTC that is being risked.
2. `h00.ooo`: The oracle, in this case https://h00.ooo
3. `/s/EPL/match/2021-10-16/LEI_MUN.vs=MUN_win`: The event the proposal is for. 
   This can be read as "Whether Manchester United beats Leicester City in their EPL (English Premier League) match on 2021-10-16".
   Note that `gun` can only bet on events with two possible outcomes.
4. `ৠöഓཀসƵ୯รແྋŸǢঊಹహĈၐ೨ø໕Ǜœબ೮ǇЧ༃ཆɁঢӛႱ၃ಢͱธྈͰଢƿɧƶവϳॻŹဪ໖ƸØ`: cryptographic input to the bet. A public key and list of inputs.

## Making your offer

When you've found a proposal you want to bet on make an offer to it.
For the proposal above, you have to decide whether you think Manchester United will win or not and how much you are willing to risk.

Run the offer command like so:

```
gun bet offer 0.005#h00.ooo#/s/EPL/match/2021-10-16/LEI_MUN.vs=MUN_win#ৠöഓཀসƵ୯รແྋŸǢঊಹహĈၐ೨ø໕Ǜœબ೮ǇЧ༃ཆɁঢӛႱ၃ಢͱธྈͰଢƿɧƶവϳॻŹဪ໖ƸØ
```

You'll be prompted to choose:

1. Which outcome you want to bet on. In this case `true` if you think Manchester United will win otherwise `false`.
2. How much you want to risk to get the 500,000 sats on offer in the proposal.
   You can bet as low as 1000 sats to give yourself 1 to 500 risk reward ratio. 
   [@GoUpNumber] will always take it if they don't get any better offers.

Copy the [base2048] gibberish you get as output and paste it somewhere the proposer can see it e.g. reply to the tweet or send a direct message to whoever made the proposal.

> 💡 Consider posting the offer under a colorful pseudonym so not even the proposer knows who they're betting with.

### Changing the default fee

As the offerer you choose the fee that the bet transaction will pay.
The default fee `gun` chooses at the moment is quite high (it attempts to get it into the next block i.e. `--fee in-blocks:1`).
This is inappropriate for most offers unless they are being made last minute.
For example, You can use `--fee` to set the fee so it should confirm some time in the next 30 blocks at the current fee rate.

```
gun bet offer --fee in-blocks:30 0.005#h00.ooo#/s/EPL/match/2021-10-16/LEI_MUN.vs=MUN_win#ৠöഓཀসƵ୯รແྋŸǢঊಹహĈၐ೨ø໕Ǜœબ೮ǇЧ༃ཆɁঢӛႱ၃ಢͱธྈͰଢƿɧƶവϳॻŹဪ໖ƸØ
```

## Monitor you offer

The easiest way to monitor the offer is to do:

```
gun -s bet list
```

Immediately after it will be in the `offered` state.
When/if the proposer has taken your offer it will be in the `unconfirmed` state and then the `confirmed` state when it ends up in the chain.
If the proposer chooses a different offer it will end up in the `canceled` state (or if you use [`gun bet cancel`](./cancel.md) to cancel it).

Take note of how long there is until whatever you are betting on takes place.
If the bet is not in the `confirmed` state a just prior to the event starting you should run `gun bet cancel <bet-id>` where bet id is the number in the first column of the `gun bet list` output.
If you are forced to manually cancel your offer you should consider avoiding betting with that proposer again as it is bad manners for a proposer to leave this to you.

## Claiming your winnings

If it doesn't get canceled, eventually your bet should go from the `confirmed` to the `won` or `lost` state.
Just keep running `gun -s bet list` to check this.

If you go into the `lost` state you don't need to do anything further.
Thank you for your sacrifice.

If you go into the `won` state you can use `gun bet claim` to put the funds back into your main wallet.
Alternatively, you can just use `gun bet send <amount> <address>` to send coins to an address which will always spend any won bets you have open.

There is no strict time limit to claiming won bets but if you have data loss while the bet is in the `won` state the coins will be difficult to recover (See [backup and recovery](../backup-and-recovery.md)).

## Troubleshooting 

### Oracle hasn't attested


If it's taking longer than expected have a look whether the oracle has attested to the event.
The easiest way to find it on [outcome.observer] but you can also manually curl the URL. For example:

```
curl -sL https://h00.ooo/s/EPL/match/2021-10-16/LEI_MUN.vs=MUN_win | jq
```

Should have a `attestation` field in the output.

> ⚠️ In the current protocol, if the oracle disappears of fails to ever attest to the event you have both lost your coins.
> The oracle `h00.ooo` is run by [LLFourn](https://github.com/LLFourn) who wrote this document so it might be best to stick to that for now.


### The Oracle has attested to the wrong thing

You can blame the operator of the oracle.

### Something else

Report this as a bug on [GitHub](https://github.com/LLFourn/gun) or contact LLFourn in some other way.

[@GoUpNumber]: https://twitter.com/GoUpNumber
[base2048]: https://docs.rs/base2048
[outcome.observer]: https://outcome.observer