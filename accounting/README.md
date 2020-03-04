# Accounting protocol

Interoperable Accounting for Exchange Communities

## Introduction
The Accounting API defines a protocol to make payments and charges between members of exchange communities. It can be used for simple local transaction between members of the same exchange group but it also defines a way to make payments across different exchange groups, exchanging the currencies. That makes the protocol very powerful and different from most other options out there.

The protocol is very to implement and understand: just a regular JSON RESTful API following the [JSON:API](https://jsonapi.org) guidelines. It is also not tight to any technology. However it makes use of standard cryptography techniques to make the protocol secure and minimize the trust the members must have with the system. Concretely, members only have to trust their local exchange groups. Local exchange groups must only trust their local neighbours (the currencies they are directly connected to). The system allows then transactions with remote currencies making several currency exchanges from the origin to the destination.

It does not use a shared ledger and it doesn't use any global consensus mechanism. That allows infinite scalability and also it allows a high level of privacy even for transactions with remote exchange groups.

Designed specifically for community exchange groups. It does not use the concept of external market maker to exchange between different currencies. The exchange groups themselves enable gateways that connect their currencies only with their neighbour currencies (one of them is enough) or with one clearing central at the exchange rate they choose. Exchange groups must actively take care of their balance trades with their directly connected currencies for the health of their own currency, trying to keep it close to zero. In a simple setting with only one gateway to remote currencies, that means that the total amount bought outside the exchange group must be roughly equal than the total amount sold outside the group.

## Objects
These are the different objects or resources in the API.

### Currency
Defines a currency. The 4-letter codes used in CES (https://community-exchange.org) and IntegralCES (https://integralces.net) are of `CEN` type. Other types of codes may be defined. 

The `scale` is the integer such that `10^(-scale)` is the minimal division for that currency. Currency amounts are specified in this API as integer multiples of `10^(-scale)` minimal unit. `decimals` is the number of decimal digits that should be usually presented to the user.

The `value` is important field for exchanging with other currencies. It defines the value of that currency against a global virtual value that is defined to be an average basic human hour of labor divided by 10^6. So, if the value of currency is exactly on hour of labour (for timebanks, for example), the value field will be 10^6. If the value of a currency is paired to, for example, the Euro, then the value field will be around 10^5. In practice, once the first currency has set their `value` field, new currencies are setting their value compared to previous currencies.

```JSON
{
    "type": "currencies",
    "id": "XXXX",
    "attributes": {
        "code-type": "CEN",
        "code": "WDLD",
        
        "name": "wonder",
        "name-plural": "wonders",
        "symbol": "₩",
        "decimals": 2,
        
        "scale": 4,
        "value": 100000,
    }
}
```
### Account
An account holds a balance of a specific currency and is the source or destinatin of money transfers.

The credit limit is the maximum balance the account may have, and the debit limit the minimum negative balance. A negative value means no limit. These limits are set by the exchange group administration.

`capabilities` are the operations allowed from this account. They are:
 - `pay`: The account may initiate payments.
 - `charge`: The account may initiate charges.
 - `connector`: The account may act as a *direct connector* for extern payments.
 - `rippling`: The account may act as a *rippling connector* for extern payments.

The key may or may not be unique for the account. In centralized systems where all accounts are managed by a single server, all accounts may share the same cryptographic key.

```JSON
{
    "type": "accounts",
    "id": "XXXX",
    "attributes": {
        "code": "Alice",
        "balance": 3200000,
        "locked": 0,
        "credit-limit": -1,
        "debit-limit": 5000000,
        "capabilities": ["pay", "charge"],
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
    },
    "links": {
        "self": "https://komunitin.org/WDLD/accounts/Alice"
    }
}
```
### Transfer
A transfer is a plain JSON object defining a movement of certain amount of credit from one account to another.

```JSON
{
    "payer": "https://komunitin.org/WDLD/accounts/WDLD0002",
    "payee": "https://komunitin.org/WDLD/accounts/WDLD0003",
    "amount": 200000,
    "meta": "10 kg of potatoes"
}
```
#### Meta
The meta field may be used to carry any information related to the transfer. In its simplest form, it is just a plain string with a description of the transfer.

If not a string, then the `meta` field is an object with a field `type` that identifies the scheme of the meta object.

The protocol may be extended with different types for the meta field, including encrypted content for better privacy. That will be relevant for extern transfers.

### Transaction
A transaction is the unit of change in the Accounting API. A transaction contains typically just one transfer, but servers may add additional transfers in a transaction to implement features such as taxes. All transfers in a transaction will be commited all or none atomically, and in the specified order.

A transaction may be in different states: 
 - `new`: The transaction has been created and passed the automatic validations.
 - `pending`: The transaction is awaiting acceptance by at least one party.
 - `accepted`: The transaction is accepted by all parties and ready to be committed.
 - `committed`: The transaction has already been committed. Once a transaction is committed, it can't be undone.  
 - `rejected`: The transaction has been rejected by one of the parties and won't be committed.

 The `expires` field has different meanings depending on the transaction state. For a `new` transaction, it means the time where it may be automatically deleted. For a `pending` trnsaction, the miximum time it should be `accepted`or `rejected` before automatic behavior (which may be automatic rejection or acceptance). For an `accepted` transaction, the maximum time where it should be committed. 

```JSON
{
    "data": {
        "id": "uuid2",
        "type": "transactions",
        "attributes": {
            "transfers": [{
                "payer": "https://komunitin.org/WDLD/accounts/WDLD0002",
                "payee": "https://komunitin.org/WDLD/accounts/WDLD0003",
                "amount": 200000,
                "meta": "10 kg of potatoes"
            }],
            "state": "new",
            "created": "2020-08-19T23:15:30.000Z",
            "updated": "2020-08-19T23:15:30.000Z",
            "expires": "2020-08-19T23:20:30.000Z",
        },
        "relationships": {
            "currency": {
                "links": {
                    "related": "https://xchange.net"
                }
            }
        },
        "links": {
            "self": "https://xchange.net/transactions/uuid2
        }
    }
}
```

### Extern transfers

Extern transfers define a movement of amount from one account in one currency from another account in another currency, eventually managed by a remote server.

```JSON
{
    "payer": "https://wonderland.org/alice",
    "payee": "https://reggaex.org/bob",
    "amount": 20000,
    "meta": "10 kg of potatoes",
    "local-payer": "https://xchange.net/wonder",
    "local-payee": "https://xchange.net/reggaex",
    "payer-signature": "alice's",
    "payee-signature": "bob's"
}
```
Beyond regular transfer attributes, they have the `local-payer` and `local-payee`. These are the local accounts that are involved in the extern transfer. They may be ommited if are equal to `payer`or `payee` respectively.

Extern transfers may also carry the cryptographic signature of the payer and the payee.

#### Signature algorithm
In order to build or verify a transfer signature the following string must be created:

```
[payer]:[payee]:[amount]:[meta]:[state]:[date]:[nonce]
```

Where the first 4 elements are the content of the transfer fields, `date` is the transaction `updated` field and `nonce` is a random number also included as a transaction field. Note that the signature includes the state, so a s signature will be used as a proof of acknowledgment of the state of a transaction. 

Encode the string in UTF-8. Then use the private key of the signing account and the cryptographic algorithm specified in the account key and sign the transaction using a standard algorithm. Finally encode the result in BASE64.

In order to avoid replay attacks, servers should persist all transactions for at least one month and reject any signature with repeated nonce and also reject all signatures older than one month. This way a signature can't be reused for a new transaction.

## Endpoints
The API follows the [JSON:API](https://jsonapi.org) guidelines.

The common CRUD operations are defined at `currency` and `accounts` endpoints for administrative operations.

Payments and charges are done using the `transactions` endpoint. Tipically a transaction will be created with a `POST` request to the transactions endpoint and later it will be either accepted, commited or rejected by updating the transaction using `PATCH` requests that change the `state` field and eventually the signature fields.

## Extern payments

### Introduction

This accounting protocol defines how to perform credit transfers between accounts in remote exchange groups, with different currencies, eventually managed from different servers. In order to do so, the protocol makes successive exchanges from the *source account* to the *destination account* through a chain of connections. A *connection* between two currencies is a pair of accounts, one in each currency and each one owned and automatically managed by the other currency. These type of accounts are called *connector accounts* or just connectors. Connector accounts in one currency represent the balance of trade with other currency.

For example, suppose that Alice has an account in currency α and Bob has an account in currency β and Alice wants to pay 10β to Bob. Suppose there is a connector B in currency α and a connector A in currency β. B is owned by the β management system but belongs to α currency and therefore subject to their administration. Conversely, A is owned by α but subject to β rules. Suppose also that the exchange rate between α and β is 1α = 2β. The transaction from Alice to Bob will be then transformed to two local transactions:

`Alice ---(5α)--->B` and `A ---(10β)--->Bob`

In the previous example there were a connection between the two currencies. However the protocol allow transactions between currencies that are not directly connected through a chain of connections. A `rippling connector` is an account that allows automatic routing of transactions that do not end nor start in the currency they represent.

For example, suppose that now Alice wants to pay 10γ to Carol, where 10α = 1γ, but α and γ are not directly connected. Imagine however that the α-β connection still exists and that there is a β-γ connection formed by accounts C in β and B' in γ. Then the transaction would be:

`Alice ---(100α)--->B` and `A ---(200β)--->C` and `B' ---(10γ)--->Carol`

In this case, accounts B and B' are rippling connectors because they participate in a transaction that doesn't start nor end in β.

### Currency stability

Currencies are free to establish the connections they want with other currencies. However, the currency administration must be aware that exchange with other currencies can be a source of inestability for their currency if it is not well balanced. Rippling is even more dangerous since the trade balance changes without any local action. It is important then to properly set the credit and debit limits to connector accounts and to actively make policies to keep the balances of trade close to zero.

For example, several currencies from a region can create a virtual currency, connect all to this currency and agree on some rules for the connector accounts in this virtual currency (credit and debit limits). They will then all be able to trade between them and each group will only have to take care of the balance of their single connector account.

### Protocol

The protocol for extern payments is an extension of the protocol for local payments. The idea of the protocol is that any exchange group may establish trust with one or a few other currencies and create connections with them. With the help of these trusted local connections and a cryptographic signature, we may securely extend transactions through a global network.

1. `POST` or `PATCH` a transaction with a remote destination account.
2. Verify that the transaction si valid and can be applied.
3. If source account is local, add the signature of source account, otherwise verify the source signature.
3. Choose the best connector to route this transaction
4. Login to the connector account in an extern currency and perform the transaction.
6. If the destination account is local, add the signature of the destination account otherwise verify destination signature.
5. Apply the local transaction.


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
            "balance": 3200000,
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
            "amount": 200000,
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
            "amount": 200000,
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
            "amount": 200000,
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
            "amount": 200000,
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
            "amount": 4000000,
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
            "amount": 400000,
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
 



