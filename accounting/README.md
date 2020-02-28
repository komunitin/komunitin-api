# Accounting API
## Goals
 - Easy for the client app. HTTP/RESTful interface. Support for synchronous call for the client.
 - Integrated protocol for intra-exchange and inter-exchange.
 - The local exchange groups administration should be the points of decision.
 - Currency exchange without speculation.
 - Not tight to a specific implementation or ledger.
 - Compatible with privacy.
 - Easy for exchange groups to manage (not need to be a hacker not an expert in macroeconomics not need to have contact with lots of other exhcange groups).
 - Support for asynchronous comunication between servers.

## Objects

### Currency
Defines a currency and the rules for exchange with that currency. The base value is an average basic human hour of labor divided by 10^6.
```JSON
{
    "type": "currencies",
    "id": "XXXX",
    "attributes": {
        "value": 100000,
        "symbol": "₩",
        "code": "WDLD",
        "code-type": "CEN",
        "decimals": 2,
        "default-debit-limit": 0,
        "default-credit-limit": -1
    }
}
```
### Account
Defines an account that holds a balance of a specific currency and is the source or destinatin of money transfers. In centralized systems where all accounts are managed by a single server, all accounts may share the same cryptographic key.

```JSON
{
    "type": "accounts",
    "id": "XXXX",
    "attributes": {
        "code": "Alice",
        "balance": 32000,
        "sequence-number"?: 321,
        "credit-limit": -1,
        "debit-limit": 50000,
        "scale": 2
    },
    "relationships": {
        "currency": {
            "links": {
                "related": "https://komunitin.org/WDLD/currency"
            }
        },
        "keys": [
            {"data": {"id": "xyz", "type": "RSA2048"}}
        ]
    }
}
```
### Transfer
A transfer is a JSON object defining a movement of certain amount of credit from one account to another. The meta field is an arbitrary application-specific JSON object. The amount is expressed in payee currency and represents the exact amount they will receive after the transaction.

```JSON
{
    "payer": "https://komunitin.org/WDLD/accounts/WDLD0002",
    "payee": "https://komunitin.org/WDLD/accounts/WDLD0003",
    "amount": 2000,
    "scale": 2,
    "currency": "https://komunitin.org/WDLD/currency",
    "description": "10kg of red potatoes",
    "meta": {},
}
```
### Local transfer
Is a transfer where both payer and payee belong to the same currency.

### Transaction
A transaction is the unit of change in the Accounting API. It is made of one or several transfers which will be commited atomically and in the order speciifed. 

A transaction may be in different states: `new`, `prepared`, `accepted`, `commited`, `rejected`.

## Payment flow
A payment or payment request from a user A to a remote user B may involve just a single local operation or a chain of multiple exchange operations along different servers.

:ghost: Briefly describe the ripple scheme applied to this case.


Payments or payment requests are done in two phases. The first phase is the *prepare phase* and the second is the *commit phase*. This way all nodes can perform as much validation as they need during the *prepare phase* and can reject transactions before any money has started to move between accounts. Also, this first phase allows to share information between the two ends of the chain.

## Payment flow example

This is an example of a payment flow through the federation protocol. Imagine that Alice wants to pay to a remote account held by Bob. Alice has an account in Wonderland exchange (₩) and Bob has an account in ReggaeEx exchange (₽). Wonderland and ReggaEx don't have any way to directly trade, but they all know the intermediate exchange, XChange (₡). Concretely, each exchange owns an account in their respective neighbour exchanges.

| Account   | Exchange | Owner |
|-----------|----------|-------|
|`https://wonderland.org/alice`  | Wonderland (₩)   | **Alice** |
|`https://wonderland.org/x`      | Wonderland (₩)   | XChange   |
|`https://xchange.net/wonder`  | XChange (₡)      | Wonderland|
|`https://xchange.net/reggaex`  | XChange (₡)      | ReggaEx   |
|`https://reggaex.org/x`         | ReggaEx (₽)      | X         |
|`https://reggaex.org/bob`       | ReggaEx (₽)       | **Bob**   |

With this setting, a payment from Alice to Bob would go as follows:

### 1. Information
Alice client app gets Bob account and currency public details, so it can show her further details before initiating the transfer. It should also fetch the exchange value for ₽ and ₩, so it can calculate the exchange rate. This is public information.

```HTTP
GET https://reggaex.org/bob
```
```JSON
{
    "data": {
        "id": "uuid",
        "type": "accounts",
        "attributes": {
            "code": "WDLD0002",
            "balance": 32000,
            //...
        }
    }
}
```

### 2. Prepare transaction
Alice Client app posts a transfer JSON to the `transactions` endpoint in order to get a transaction. The "id" must be set by the client in order to avoid duplicate transactions on retry, using a UUID generator.

This call will not apply the transaction but will rather setup the system so it's ready to commit the transaction and will return a transaction with other details included.

```HTTP
POST /transactions HTTP/1.1
Host: wonderland.org
```
```JSON
{
    "data": {
        "id": "uuid1",
        "type": "transactions",
        "transfers": [{
            "payer": "https://wonderland.org/alice",
            "payee": "https://reggaex.org/bob",
            "amount": 2000,
            "scale": 2,
            "currency": "RGEX",
            "description": "bla bla bla"
        }]
    }
}
```

### 3. Prepare transaction: first hop
The services does some validation: Alice has sufficient balance, etc. However, the payee account belongs to a remote currency that, in this case, is untrusted by the Alice's currency. But Alice's exchange has a gateway to possibliy reach Bob: XChange. The Wonderland (Alice's) server makes a *Prepare pransaction* call to XChange server, changing the Alice's acount to the virtual Wonderland exchange account in xchange.net.

Wonderland server signs the transaction using Alice's key, so if the transaction finally reaches Bob, he can be sure where the request comes from and how much she originally wanted to transfer.

```HTTP
POST /transactions HTTP/1.1
Host: xchange.net
```
```JSON
{
    "data": {
        "id": "uuid2",
        "type": "transactions",
        "transfers": [{
            "source": "https://wonderland.org/alice",
            "payer": "https://xchange.net/wonder",
            "payee": "https://reggaex.org/bob",
            "amount": 2000,
            "scale": 2,
            "currency": "RGEX",
            "description": "bla bla bla"
        }],
        "source-signature": "alice(source, destination, amount, new)"
    }
}
```
### 4. Prepare transaction: second hop.
XChange can't directly execute the transaction since, again, the payee account is remote. However XChange has a gateway with Bob's currency. So after validating the local part of the transaction, it makes the `transactions` POST to the final service, its their gateway account.

```HTTP
POST /transactions HTTP/1.1
Host: xchange.net
```
```JSON
{
    "data": {
        "id": "uuid3",
        "type": "transactions",
        "transfers": [{
            "source": "https://wonderland.org/alice",
            "payer": "https://reggaex.org/x",
            "payee": "https://reggaex.org/bob",
            "amount": 2000,
            "scale": 2,
            "currency": "RGEX",
            "description": "bla bla bla"
        }],
        "source-signature": "alice(source, destination, amount, new)"
    }
}
```
### 5. Prepare transaction: final
The final exchange ReggaeEx validates the transaction and, since Bob's account is configured to automatically accept any incomming payment, it returns the transaction ready to be commited.
```HTTP
201 Created
```
```JSON
{
    "data": {
        "id": "uuid3",
        "type": "transactions",
        "transfers": [{
            "source": "https://wonderland.org/alice",
            "payer": "https://reggaex.org/x",
            "payee": "https://reggaex.org/bob",
            "amount": 2000,
            "scale": 2,
            "currency": "RGEX",
            "description": "bla bla bla"
        }],
        "created": "2020-08-19T23:15:30.000Z",
        "expires": "2020-08-19T23:20:30.000Z",
        "state": "accepted",
        "source-signature": "alice(source, destination, amount, new)",
        "destination-signature": "bob(source, destination, amount, accepted)"
    }
}
```
ReggaeEx gives 5 minutes to the caller for the transaction to be fulfilled.

### 6. Prepare transaction: return

XChange gets the accepted transfer from ReggaeEx above. The `destination-signature` allows XChange to prove to Wonderland that Bob will actually receive 20₽.

```HTTP
201 Created
```
```JSON
{
    "data": {
        "transfers": [
        {
            "source": "https://wonderland.org/alice",
            "destination": "https://reggaex.org/bob",
            "payer": "https://xchange.net/wonder",
            "payee": "https://xchange.net/reggaex",
            "amount": 4000,
            "scale": 3,
            "currency": "XCHG",
            "description": "bla bla bla"
        }],
        "created": "2020-08-19T23:15:31.000Z",
        "expires": "2020-08-19T23:20:00.000Z",
        "state": "accepted",
        "source-signature": "alice(source, destination, amount, new)",
        "destination-signature": "bob(source, destination, amount, accepted)"
    }
}
```
XChange substracts 30 seconds to the expiry time to be sure it has time to fulfill the upstream transaction if needed.

### 6. Prepare transaction: response
Wonderland returns to Alice client the final local transfer. Wonderland substracts an additional minute to the expiry time.
```HTTP
201 Created
```
```JSON
{
    "data": {
        "id": "uuid1",
        "transfers": [{
            "destination": "https://reggaex.org/bob",
            "payer": "https://wonderland.org/alice",
            "payee": "https://wonderland.org/x",
            "amount": 4000,
            "scale": 2,
            "currency": "WDLD",
            "description": "bla bla bla"
        }],
        "created": "2020-08-19T23:15:32.000Z",
        "expires": "2020-08-19T23:19:00.000Z",
        "state": "accepted",
        "source-signature": "alice(source, destination, amount, new)",
        "destination-signature": "bob(source, destination, amount, accepted)"
    }
}
```
Alice now knows that the payment to Bob is possible and that it will cost 40₩ to send 20₽ to Bob.

### 8. Commit
Now she has two options, either to fulfill the transaction to actually pay the amount or reject the transaction. The second option is a `DELETE` request to `transactions/uuid1`, that will be propagated to subsequent `DELETE` requests ahead in the chain. 

In case she wants to fulfill the transaction, she sends the request:

```HTTP
PATCH /transactions/uuid1
Host: wonderland.org
```
```JSON
{
    "data": {
        "state": "committed"
    }
}
```
Wonderland server will update the `source-signature` field:
```JSON
{"source-signature": "alice(source, destination, amount, commited)"}
```

### 9. Commit propagation
Wonderland server needs to do two things in order to commit this transaction:
- To transfer 40₩ to `https://wonderland.org/x`
- Send the commit request to the upstream transaction.

In order to do the two operations atomically, the server should:
1. Verify `source-signature`
2. Lock the funds from the payer account
3. Make the upsteam request and wait for the response
4. Verify the destination signature
5. Commit the local transfer
6. Return the response

```HTTP
PATCH /transactions/uuid2
Host: xchange.net
```
```JSON
{
    "data": {
        "state": "committed",
        "source-signature": "alice(source, destination, amount, commited)"
    }
}
```
XChange server will do the same to ReggaEx that hopefully will actually commit the last transaction, returning a `200 OK` with the commited transaction and the destination signature updated.

```HTTP
201 Created
```
```JSON
{
    "data": {
        "id": "uuid1",
        "type": "transactions",
        "transfers": [{
            "source": "https://wonderland.org/alice",
            "payer": "https://reggaex.org/x",
            "payee": "https://reggaex.org/bob",
            "amount": 2000,
            "scale": 2,
            "currency": "RGEX",
            "description": "bla bla bla"
        }],
        "state": "commited",
        "commited": "2020-08-19T23:16:00.000Z", 
        "created": "2020-08-19T23:15:30.000Z",
        "source-signature": "alice(source, destination, amount, commited)",
        "destination-signature": "bob(source, destination, amount, commited)",
    }
}
```

The `destination-signature` is the proof that Bob got the money.

### 9. Commit propagation: response
Finally the wonderland server returns the commited transaction to Alice app so the transaction is completely applied.

```HTTP
201 Created
```
```JSON
{
    "data": {
        "id": "uuid1",
        "type": "transactions",
        "transfers": [{
            "payer": "https://wonderland.org/alice",
            "payee": "https://wonderland.org/x",
            "destination": "https://reggaex.org/bob",
            "amount": 4000,
            "scale": 2,
            "currency": "WDLD",
            "description": "bla bla bla"
        }],
        "state": "commited",
        "commited": "2020-08-19T23:16:02.000Z", 
        "created": "2020-08-19T23:15:30.000Z",
        "destination-signature": "bob(source, destination, amount, commited)",
    }
}
```
Alice can now be sure that Bob got its 20₽ because of the signature.

## Fail cases
### Rejection or fail in prepare
During the prepare phase, if any intermediate node or the final one don't accept the transaction, it should return the transaction with `rejected` state and a `rejection-message`. The rejection message must be propagated downstream.
```HTTP
200 OK
```
```JSON
{
    "id": "uuid1",
    "type": "transactions",
    // transfers
    "state": "rejected",
    "rejection-message": "Insufficient funds",
    "rejection-code": "1001"
}
```
In other cases of error, such as upstream timeout, the server may return `50X` and an [error object](https://jsonapi.org/format/#error-objects) that will be equally propagated downstream.

### Prepare timeout
:gost: TODO

### Rejection or timeout in commit
Failures in commit phase are minimized since the commit only happens after a successfully prepare phase. However they may sporadically happen.

If an upstream node does not return or returns an error or the return message is not valid (for example, the signatures are invalid), then the caller must understand that the transaction has not been commited and return error. If the transaction was actually commited upstream, only the failing node "loses money" on their upstream node.

### Fraud

//TODO

# Checklist
There are issues that need to be solved, but I think all of them are solvable, either by documentation or by slightly changing the protocol.

 - Add support for manual acceptance
 
 TODO

 - how can Bob be sure that the money comes from Alice?
 
 She can check the commit source-signature field.

 - Intermediate servers can access to description and other info that's irrelevant to them and possibly delicate.

Extend the protocol by adding encrypted data into the `meta` transfer field, while having empty or dummy description.

 - What if source/destination/intermediaries want to charge a fee for the transaction or add metadata?

They may do so by applying the exchange rate they want. Nodes may allow (local) competence among different paths. A node will tipically chose the path that most benefits them. They may add supplementary transfers into their object.

 - How can Alice be sure that the money actually arrive to Bob?

She can check the commit destination-signature.

- Add support for asynchronous API
Todo: Either use a callback strategy or use the notificatin API, since jsonapi recommendation is not well suited for chains of dependent requests. Take however the response codes and object structure from jsonapi recommendation.

 - What happens if an intermediary fails?
 - What happens if intermediary actively commits fraud?
 - What happens if balances change between.
 - What happens when intermediate node times out after propagating the commit?
 - Lock balances at prepare stage?
 - Now the trust is not really local, since, if the final node behaves badly, alice still spends the money even if first and second node behave good. This MUST (and can!) be fixed (hashlock!).
 - What happens if alice don't fulfill nor delete the transaction?
 - Missing async support.
 



