# Hoard
Bitcoin vaults without a soft fork

# Proposal

```
Create two private keys, Key1 and Key2
Create utxo X which is locked to Key1 -- this is the "vault"
Create (but do not sign) tx1 which spends utxo X to create utxo Y -- this is the "midstate"
Utxo Y (the midstate) has 2 spending paths:
(a) Key1 and Key2 can (together) spend the money immediately
---this allows for a "revaulting tx," tx2, explained below
(b) Key2 can spend the money after a timelock
---this allows for an "unvaulting tx" that sends the money wherever you want
Create and sign tx2 which spends Y according to the logic in (a)
---tx2 should send the money to utxo Z
---utxo Z should be a clone of utxo X and the subsequent logic
---except with a different Key1 and Key2
Sign tx1
Delete Key1
Save tx1, tx2, and Key2
```

# How it works

Until you broadcast tx1, your money is "in a vault." Tx1 locks your money in a "midstate" -- utxo Y -- where only two things can happen: either you broadcast tx2, a "revaulting transaction" that just puts it into another vault that is identical to the first one, or you wait for a timelock to expire and then use your money however you like.

Here is how this vault setup protects you from thieves: while your money is in the vault, a thief cannot take it because they cannot get Key1. You deleted Key1, so *you* don't even have it. The only things a thief can steal are your presigned transactions (tx1 and tx2) and Key2. Key2 does the thief no good by itself: they cannot use it to spend your money unless they first broadcast tx1 -- which puts your money in the midstate -- then wait for a timelock to expire, and hope you don't revault your money in the meantime via tx2, thus blocking the theft.

Also, if you can't be sure you'll know when tx1 is broadcast, you can let a non-custodial watchtower do it for you. If they see tx1 was broadcast, they can alert you via email or similar and ask you to confirm within a day or so that you sent tx1. If you don't confirm it was you, they can broadcast your revaulting transaction on your behalf. Whereas if you *are* the one who sent tx1, you can just wait for the timelock to expire (and, if you use a watchtower, confirm for them that you are the unvaulter), then spend the money however you like via Key2.

If you make the timelock 6 months long and create 100 copies of this contract where each "utxo Z" is just another copy of this contract, then you have a vault that can withstand 50 years of fraudulent unvaulting attempts.

# Answering objections

James O'Bierne's [op_vault whitepaper](https://jameso.be/vaults.pdf) has a section on "Precomputed vaults" on page 2. This is mostly designed to cover the tradeoffs of Bryan Bishop's "[python vaults](https://github.com/kanzure/python-vaults)" implementation of vaults, which is similar to this one except in Bryan's model, utxo Z locks the money to a bitcoin address owned by a very-cold-storage "recovery wallet" instead of another copy of the vault. Most of the tradeoffs James discusses apply to my vault model as well so I will address them here.

> Key deletion cannot be proved, so ensuring that the vault isn’t backdoored is more difficult.

This seems solvable to any desired degree by using an N of N multisig to create the vault. If you are one of the signers and you delete your key, you can be certain the vault isn't backdoored.

> The custodian must not lose the presigned transactions, since there is no other way of spending the bitcoin.

In my mental model, once you create a presigned transaction and throw away the key, the presigned transaction essentially *becomes* the private key. Like a private key, a presigned transaction is a piece of text that lets you spend some money. If bitcoin had covenants, the private key to a covenant-encumbered utxo would only *work* by forcing you to create predetermined outputs when you use it. That is also true with a presigned transaction: it works but it forces you to create predetermined outputs when you use it.

Since I view presigned transactions as equivalent to the private keys of covenant-encumbered outputs, I read this sentence as essentially equivalent to this complaint: "Covenants are bad because if you lose your private keys you're totally screwed." -- "Presigned covenants are bad because if you lose your presigned transactions you're totally screwed." The answer is the same for both: the security model is "take care not to lose that piece of text."

> The sensitive data that is necessary to store indefinitely grows linearly with the number of vaults created.

Yes but this "problem" does not rise to signfiicance in my opinion. The data is a fixed amount per utxo, basically 200 presigned transactions per utxo. If you have millions of utxos, you might have to buy a hard drive, but if you're an exchange, that's not a significant addition to your operating expenses, especially for an optional feature. Not all users will want their utxos to be timelocked for 6 months before they can withdraw them so not all users of an exchange will want a vault.

> The spend target for the vault is static, and presumably must correspond to some kind of hot (or “warm”) wallet.

I'm not sure that my model corresponds exactly to Bryan Bishop's but mine has a "mid state" where a certain public key can send the money anywhere after a timelock expires if the money was not "revaulted." If he is talking about the timelocked pubkey, that pubkey does not have to be hot or warm. It can be very very cold, created on the most offline device ever and only its pubkey exported for use in the vault contract. It's totally up to the user how cold that key is.

> Loss of control of that hot wallet necessitates sweeping all presigned transactions to the likely difficult-to-access recovery wallet.

This seems like a strange objection. "What if an unauthorized person gets your keys" is the whole reason vaults exist, and they offer this solution: if an unauthorized person tries to unvault your money, a timelock gives you an opportunity to stop them by revaulting your money. I think Bryan Bishop's model also lets you send the money to an "even colder" recovery wallet but I don't need to defend someone else's model.

> Arbitrary vault withdrawal amounts are not possible after the structure of the vault is locked in by presigning.

That is true but there is a decent mitigation: vault many small amounts. If you do that then you can unvault an arbitrary amount by unvaulting as many of the small amounts as you need to add up to the arbitrary amount. This has a tradeoff though because you have to pay a fee for each amount you want to unvault.

> Vault operations, namely recoveries and unvaultings, cannot be batched together

This is mostly true, I don't know a good mitigation for this and it is unfortunate. So far this seems like the only meaningful tradeoff.

> Once initiated, other deposits cannot (or should not) be sent to after the inital deposit.

I think this is equivalent to saying "but I want to reuse my vault address." That seems like a trivial non-problem. Just create a second vault address. Non-problem solved.

> This prevents e.g. exchanges from initiating vaults and then letting customers deposit directly into vaults at-will.

How? Just give the user a url to click where you generate a new vault address for them. Bam, non-problem solved.
