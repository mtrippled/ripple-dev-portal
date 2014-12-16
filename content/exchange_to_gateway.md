# Becoming a Ripple Gateway #

An existing online financial service, such as a payment system or digital currency exchangea, can provide additional value to customers by acting as a Ripple Gateway. This provides users the ability to send cross-currency payments to users linked by other Ripple Gateways, and potentially provides a new revenue source for balances deposited, withdrawn, or transferred in Ripple.

Expanding an existing exchange system to support Ripple is a relatively simple task. This document explains the concepts necessary to set up a system, and covers the details of doing so. In this document, we use a fictional online currency exchange named "ACME" and its users as examples of how ACME can expand its business to include being a Ripple Gateway.

## Ripple Gateways Explained ##

A Ripple _*Gateway*_ is an entity that exchanges balances in the Ripple Network for balances in the Ripple Network -- in other words, performing deposits and withdrawals from Ripple. Typically, a Gateway holds money (or other assets of value) outside of Ripple, and creates _*issuances*_ in the Ripple Network to represent them. Within Ripple, issuances can be sent, traded, and exchanged atomically without the gateway's intervention. 

Ripple's native cryptocurrency, XRP, can also be exchanged for issuances, and sent to any Ripple account, but exists only within the Ripple Network and cannot be created, deposited, or withdrawn the way that issuances can. All Ripple accounts need a small reserve of XRP in order to pay the network costs of maintaining their accounts and sending transactions. Some users may choose to hold additional XRP to use as a means of exchange, but large holdings of XRP are not strictly necessary to operate a gateway.


### Ripple Trust Lines and Issuances ###

A "trust line" is link between two accounts in Ripple that represents an explicit statement of willingness to hold gateway debt obligations. Those debt obligations, which we call *issuances*, are digital assets that can be sent and traded. A *gateway deposit* is when someone sends value (outside the Ripple Network) to a gateway in exchange for issuances on the Ripple network. An issuance represents a promise that whoever holds the issuance can redeem it with the issuer on the Ripple Network for that amount of value (off the Ripple Network). Doing so is a *gateway withdrawal*. 

All assets in Ripple, except for the native cryptocurrency XRP, are represented as issuances. Issuances get their value from the promise that the gateway issuing them will honor the obligation that the issuances represent; there is no computer system that can force a Ripple gateway to honor that obligation. Therefore, trust lines ensure that users only hold issuances from gateways they trust.


### Hot and Cold Wallets ###

It is strongly recommended that Ripple gateways employ a "hot wallet / cold wallet" strategy. This enforces a separation of roles that promotes strong security. ("Wallets" in Ripple are equivalent to Accounts.)

The cold wallet should remain offline, and serves as the asset issuer. This means that the secret key to sign transactions for the cold wallet is not stored on a computer that is connected to the Internet. Periodically, a human operator creates and signs a transaction (preferably from an entirely offline machine) in order to refill the hot wallet's balance. Because the cold wallet is the account creating the issuances, customer accounts holding those issuances must trust the cold wallet. 

A hot wallet makes payments to the gateway's users in Ripple by sending them issuances created by the cold wallet. It also needs a trust line to the cold wallet. A gateway can use one or more "hot wallet" accounts, but each hot wallet has a limited balance of the gateway's issuances. If it is compromised, the gateway can only lose as much currency as the hot wallet holds. However, this means that the gateway must monitor the hot wallet's balance, so that it doesn't run out during ordinary operation.


### Trading on Ripple ###

After the issuances have been created in Ripple, they can be freely transferred and traded by Ripple users. 

- Anyone can buy/sell EUR@ACME on Ripple
    - including users who don't have ACME accounts (caveat: requireauth flag)
- Ripple users trading and sending EUR@ACME to one another requires no intervention by ACME


## Prior to Ripple Integration ##

Our example exchange, ACME, already accepts withdrawals and deposits from users using some existing system, and uses an internal accounting system to track how much balance each user has with the exchange. Such a system can be modeled simply like this:

<!-- diagram: e2g_1

Alice - €4 held
ACME - €5 held
ACME Core Accounting - Bob: €1, Charlie: €2, ACME itself: €2

Alice deposits €4

Alice - €0 held
ACME - €9 held
ACME Core Accounting - Alice: €4, Bob: €1, Charlie: €2, ACME itself: €2

-->

**Assumptions:** To integrate with Ripple, we assume that an exchange such as ACME meets the following assumptions:

* ACME already has a system to accept deposits and withdrawals from some outside payment source. 
* ACME waits for deposits to clear before crediting them internally.
* ACME always keeps enough funds on-hand to pay withdrawals on demand, subject to their terms and conditions.
    * ACME can set fees, minimum withdrawals, and delay times for deposits and withdrawals as their business model demands.
    

## Ripple Integration ##

## Deposits from Gateway to Ripple ##

A deposit into Ripple means moving funds from a user's balance at a gateway into a separate record tracking Ripple-backed funds, and then sending the equivalent amount of issuances in Ripple to the user's Ripple account.

An example of a deposit flow:

1. Alice asks to deposit €2 of her balance into Ripple
2. In its internal accounting, ACME debits Alice's balance €2 and credits the Ripple-backed balance by €2.
3. ACME submits a Ripple transaction, sending €2 to Alice's Ripple address. The €2 is marked in Ripple as being "issued" by ACME.

<!-- diagram:

ACME Core Accounting - Alice: €4, Bob: €1, Charlie: €2, ACME itself: €2, Ripple: €0
ACME hot wallet —€0→ cold wallet
Alice Ripple acct —€0→ cold wallet

Alice "deposits" €2 into Ripple (intermediate state)

ACME Core Accounting - Alice: €2, Bob: €1, Charlie: €2, ACME itself: €2, Ripple: €2
ACME hot wallet —€2→ cold wallet
Alice Ripple acct —€0→ cold wallet

ACME sends Alice's money to her Ripple account

ACME Core Accounting - Alice: €2, Bob: €1, Charlie: €2, ACME itself: €2, Ripple: €2
ACME hot wallet —€0→ cold wallet
Alice Ripple acct —€2→ cold wallet

-->

### Deposit Requirements ###

There are several prerequisites that ACME must meet in order for this to happen:

- ACME modifies its core accounting system to track money that is backing funds issued on the Ripple Network. This could be as simple as adding a record for Ripple. Optionally, a gateway can take additional steps to separate normal user funds from funds backing the gateway's Ripple issuances.
- ACME must have a Ripple account. Our best practices recommend actually having at least two accounts: a "cold wallet" account to issue currency, and one or more "hot wallet" accounts that perform day-to-day transactions. See [Hot and Cold Wallets](#hot-and-cold-wallets) for more information.
- Alice must create a trustline from her Ripple address to ACME's issuing (cold wallet) account. She can do this from any Ripple client (such as [Ripple Trade](https://www.rippletrade.com/) as long as she knows ACME's account address or Ripple Name.
- ACME must create a user interface for Alice to deposit funds from ACME into Ripple.
    - In order to do this, ACME needs to know Alice's Ripple address. ACME can have Alice input her Ripple addresss as part of the deposit interface, or ACME can require Alice to input and verify her Ripple address in advance.


## Withdrawals from Ripple to Gateway ##

A withdrawal from Ripple means moving funds from the Ripple-backed balance at a gateway into a user account in response to receiving a Ripple payment.

An example of a withdrawal flow:

1. Bob sends Ripple transaction of €1 to ACME's cold wallet
2. In its internal accounting, ACME debits its Ripple-backing balance €1 and credits Bob's balance €1.


### Withdrawal Requirements ###

In addition to the [requirements for making deposits possible](#deposit-requirements), there are several prerequisites that ACME must meet in order to process withdrawals:

- ACME must monitor its Ripple accounts for incoming payments.
- ACME must recognize the identities of users from the incoming payments.
    - We recommend that ACME should bounce any unrecognized incoming payments back to their sender.
    - Typically, the preferred method of recognizing incoming payments is through destination tags.


### Precautions ###

Processing withdrawals and bouncing incoming payments are both potentially risky processes, so a gateway should be sure to take care in implementing them. Additionally, gateways should avoid any potentially ambiguous situations when receiving customer funds. We recommend the following precautions:

- Before processing a withdrawal, make sure you know the customer's identity. This is especially important because the users withdrawing from Ripple could be different than the ones depositing.
- Understand how [Partial Payments](#partial-payments) work, and correctly read the delivered amount.


# Appendix #

## Destination Tags ##

TODO

## Partial Payments ##

TODO

## Setting Trust Lines in Ripple Trade ##

TODO