# Federated payments

Payments are the worst part of the streaming ecosystem and the area most subject to abuse. Global money pools that are split up on a per-stream basis encourage bot listeners and ultra-short content, and pooling payments across listeners means that disproportionate numbers of listens from single users will give an outsized influence on the payments sent to the artists they listen to, at the expense of other artists.

While it's out of scope to insist on a particular payment strategy (and payments are specifically *not* a part of the protocol), this document will explain a few ideas for how listeners can fairly compensate the artists that they listen to.

## Support ledger

In this system, listeners would be able to choose what "streaming rate" is fair, ideally based on a unit per time. For example, with the assumption that a typical user listens for 100 hours per month and that they would be willing to pay $10/month for their listening, their stream rate would be 10¢/hour.

The receiver then keeps track of a balance for every listener and artist. As a listener listens to music, their balance decreases, and the balance of the artists they listen to increase by the same rate.

Whenever an artist reaches a payment threshold, the system notifies the listener with the largest negative balance that they should send that artist's balance over to them. When given proof of payment, the artist's balance decreases by the transaction amount, and the listener's balance increases by the same.

## Monthly support

As a variant on the "support ledger" strategy, there could be a monthly collection of funds, and at the end of the month, any artists whose balance exceeds a payment threshold would be paid their full balance.

Anything left over would be split up among the artists with the top balances that fall below the threshold; for example, if the threshold is $5 and the coffer has $127 in it, then the top 25 unpaid artists would then receive $5.08, and their balances would be decreased by that amount (going negative, as an advance on future payment thresholds).

This strategy could also be used to allow for things like community support or ad revenue or the like to offset the costs of people who are unable or unwilling to pay.
