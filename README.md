# Hoard
Bitcoin vaults without a soft fork

# Proposal

```
Create two private keys, Key1 and Key2
Create utxo X which is locked to Key1
Create (but do not sign) tx1 which spends utxo X to create utxo Y
Utxo Y has 2 spending paths:
(a) Key2 can spend the money after a timelock
(b) Key1 and Key2 can (together) spend the money immediately
Create and sign tx2 which spends Y according to the logic in Y(b)
---tx2 should send the money to utxo Z
---utxo Z should be a clone of utxo X and the subsequent logic
---except with a different Key1 and Key2
Sign tx1
Delete Key1
Save tx1, tx2, and Key2
```

# How it works

Until you broadcast tx1, your money is "in a vault." Tx1 is an "unvaulting transaction." It locks your money in a "midstate" -- utxo Y -- where only two things can happen: either you broadcast tx2, a "revaulting transaction" that just puts it into another vault that is identical to the first one, or you wait for a timelock to expire and then use your money however you like.

Here is how this vault setup protects you from thieves: while your money is in the vault, a thief cannot take it because they cannot get Key1. You deleted Key1, so *you* don't even have it. The only things a thief can steal are your presigned transactions (tx1 and tx2) and Key2. Key2 does the thief no good by itself: they cannot use it to spend your money unless they first broadcast tx1 -- which puts your money in the midstate -- then wait for a timelock to expire, and hope you don't revault your money in the meantime via tx2, thus blocking the theft.

Also, if you can't be sure you'll know when tx1 is broadcast, you can let a non-custodial watchtower do it for you. If they see tx1 was broadcast, they can alert you via email or similar and ask you to confirm within a day or so that you did the unvaulting. If you don't confirm it was you, they can broadcast your revaulting transaction on your behalf. Whereas if you *are* the one who did the unvaulting, you can just wait for the timelock to expire (and, if you use a watchtower, confirm for them that you are the unvaulter), then spend the money however you like via Key2.

If you make the timelock 6 months long and create 100 copies of this contract where each "utxo Z" is just another copy of this contract, then you have a vault that can withstand 50 years of fraudulent unvaulting attempts.
