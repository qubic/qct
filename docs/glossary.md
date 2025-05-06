# Glossary

## TODOs / Open questions

* event data -> payload?
* Fill quorum tick data


## General

### Base terminology

* `Identity` refers to the human-readable form of the public key (60 letters, A-Z, last 4 are the checksum).
* `PublicKey` refers to the machine-readable representation of the public key (32 bytes).

### Rules

* Redundant name constructs should be avoided.
* Abbreviations should be avoided.

### Decisions

* tickNumber ... with number postfix, otherwise refers to the domain object. Postfix omitted for some properties like initialTick, incomingTick, ...
* identity/publicKey ... can be omitted, if it's clear for the API what to use (different for RPC/CLI and core code)

## Domain Objects

### Tick Info

| Name        | Description             | Aliases |
|-------------|-------------------------|---------|
| tickNumber  | Number of the tick      | tick    |
| duration    |                         |         |
| epoch       | Number of the epoch     |         |
| initialTick | First tick of the epoch |         |

### Tick Data

| Name           | Description                   | Aliases                         |
|----------------|-------------------------------|---------------------------------|
| computorIndex  |                               |                                 |
| tickNumber     | Number of the tick            | tick                            |
| epoch          |                               |                                 |
| timestamp      |                               |                                 |
| varStruct      | Deprecated. Not used anymore. |                                 |
| timeLock       |                               |                                 |
| transactionIds | List of transactions in tick  | transactions, transactionHashes |
| contractFees   | List of contract fees         |                                 |
| signature      |                               | signatureHex                    |

### Quorum Tick Data

*TODO* => @linckode

### Tick

| Name       | Comment      | Aliases |
|------------|--------------|---------|
| tickNumber | Tick number  | number  |
| epoch      | Epoch number |         |

### Entity

| Name               | Comment                                         | Aliases |
|--------------------|-------------------------------------------------|---------|
| identity           | Human readable representation of the public key | id, ID  |
| balance            | Incoming amount - outgoing amount               |         |
| incomingAmount     |                                                 |         |
| outgoingAmount     |                                                 |         |
| incomingTransfers  | Number of incoming transfers                    |         |
| outgoingTransfers  | Number of outgoing transfers                    |         |
| latestIncomingTick |                                                 |         |
| latestOutgoingTick |                                                 |         |
| tickNumber         | Tick number the information is valid for        | tick    |

### Transaction

| Name        | Comment                              | Aliases                                           |
|-------------|--------------------------------------|---------------------------------------------------|
| hash        | Unique hash of the transaction       | txId, txHash, id                                  |
| amount      |                                      | totalAmount                                       |
| source      | Sender of the transaction            | sourceId, sourceIdentity, sourcePublicKey         |
| destination | Receiver of the transaction          | destId, destinationIdentity, destinationPublicKey |
| inputType   | Type of the transaction              |                                                   |
| inputSize   | Size of the extra data               |                                                   |
| inputData   | Transaction payload                  | input, extraData, inputHex                        |
| signature   |                                      | signatureHex                                      |
| tickNumber  | Tick number of the transaction       | tickNumber                                        |
| timestamp   | Timestamp of the tick                |                                                   |
| moneyFlew   | Deprecated. Set by tx status add on. |                                                   |

### Asset

| Name                       | Comment                                        | Aliases                               |
|----------------------------|------------------------------------------------|---------------------------------------|
| issuer                     | Issuer of the asset                            | issuerIdentity, issuerPublicKey       |
| name                       | Name of the asset                              | assetName                             |
| numberOfDecimalPlaces      | Specified in issuance                          |                                       |
| unitOfMeasurement          | Specified in issuance                          |                                       |
| numberOfShares             | Total number of shares is not returned via API | numberOfUnits                         |
| pricePerShare              | Only in context of QX                          |                                       |

### Asset Issuance

| Name                       | Comment                                                       | Aliases                               |
|----------------------------|---------------------------------------------------------------|---------------------------------------|
| issuer                     | Issuer of the asset                                           | issuerIdentity, issuerPublicKey       |
| name                       | Name of the asset                                             | assetName                             |
| numberOfDecimalPlaces      | Specified in issuance                                         |                                       |
| unitOfMeasurement          | Specified in issuance                                         |                                       |
| type                       | Type of the asset information issuance, ownership, possession |                                       |
| universeIndex              | Index number (position) of the entry within the universe      |                                       |
| tickNumber                 | Tick number the information is valid for                      | tick                                  |

### Asset Ownership

| Name                  | Comment                                                       | Aliases                       |
|-----------------------|---------------------------------------------------------------|-------------------------------|
| owner                 | Entity owning the asset                                       | ownerIdentity, ownerPublicKey |
| type                  | Type of the asset information issuance, ownership, possession |                               |
| numberOfShares        |                                                               | numberOfUnits                 |
| managingContractIndex | Contract number that is managing the assets ownership         |                               |
| issuanceIndex         | Index number (position) of the issuance within the universe   |                               |
| universeIndex         | Index number (position) of the entry within the universe      |                               |
| tickNumber            | Tick number the information is valid for                      | tick                          |

### Asset Possession

| Name                  | Comment                                                       | Aliases                               |
|-----------------------|---------------------------------------------------------------|---------------------------------------|
| possessor             | Entity possessing the asset                                   | possessorIdentity, possessorPublicKey |
| type                  | Type of the asset information issuance, ownership, possession |                                       |
| numberOfShares        |                                                               | numberOfUnits                         |
| managingContractIndex | Contract that is managing the assets possession               |                                       |
| ownershipIndex        | Index number (position) of the ownership within the universe  |                                       |
| universeIndex         | Index number (position) of the entry within the universe      |                                       |
| tickNumber            | Tick number the information is valid for                      | tick                                  |

### Event

| Name            | Comment                          | Aliases            |
|-----------------|----------------------------------|--------------------|
| epoch           | Epoch number                     |                    |
| tickNumber      | Tick number                      | tick               |
| index           | Node specific event id           | eventId, id        |
| digest          |                                  | eventDigest        |
| transactionHash | Transaction the event belongs to |                    |
| type            |                                  | eventType          |
| size            | Size of the event data           | eventSize          |
| data            | Event payload                    | payload, eventData |

### Asset Transfer (event based)

| Name            | Comment           | Aliases                            |
|-----------------|-------------------|------------------------------------|
| source          | Sender identity   | sourceId, sourceIdentity           |
| destination     | Receiver identity | destinationId, destinationIdentity |
| assetIssuer     | Asset issuer      | issuerId, issuer, issuerIdentity   |
| assetName       | Asset name        | name                               |
| numberOfShares  |                   |                                    |
| transactionHash |                   | transactionId                      |
| tickNumber      | Tick number       | tick                               |
| eventType       | Event type        |                                    |

### Qubic Transfer (event based)

| Name            | Comment                 | Aliases                                                  |
|-----------------|-------------------------|----------------------------------------------------------|
| source          | Sender identity         | sourceId, sourceIdentity, sourcePublicKey                |
| destination     | Receiver identity       | destinationId, destinationIdentity, destinationPublicKey |
| amount          |                         |                                                          |
| transactionHash |                         | transactionId                                            |
| tickNumber      | Tick number             | tick                                                     |
| eventType       | Event type. Always `0`. |                                                          |

## Other

Status information: latestTick, lastProcessedTick, startTick, endTick, ...