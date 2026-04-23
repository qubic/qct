# Iteration Results

This page provides a simplified list of iteration deliverables. It is not intended to be exhaustive,
please see the respective team boards and repositories on the [qubic GitHub](https://github.com/qubic) for that.
Listed items are completed but that does not mean that all the deliverables are deployed to production yet.

## Iteration 34

Duration: April 8, 2026 - April 21, 2026

### [Integration Team](https://github.com/orgs/qubic/projects/21)

* Add log types 14 + 15 to the consumer/bridge (see [issue](https://github.com/qubic/go-log-data-publisher/issues/33)).
* Add log types 14 + 15 in elastic (see [issue](https://github.com/qubic/qubic-deployment/issues/117)).
* Add tick data dedup streams service (see [issue](https://github.com/qubic/kafka-streams-services/issues/2)).
* Add transactions dedup streams service (see [issue](https://github.com/qubic/kafka-streams-services/issues/1)).
* Improve handling of non-retriable errors in kafka publishers (see [issue](https://github.com/qubic/go-log-data-publisher/issues/19)).
* Status service: replace get document id calls with search queries (see [issue](https://github.com/qubic/go-data-publisher/issues/80)).
* Increase the response size cap (see [issue](https://github.com/qubic/archive-query-service/issues/147)).
* Investigate total connection times (see [issue](https://github.com/qubic/qubic-deployment/issues/122)).
* Fix archiver: handle disable tx status addon (see [issue](https://github.com/qubic/go-archiver-v2/issues/22)).
* Fix haproxy health check (see [issue](https://github.com/qubic/qubic-deployment/issues/121)).
* Fix asset owner pagination sorting (see [issue](https://github.com/qubic/qubic-stats-service/issues/25)).
* Fix empty ticks endpoint (see [issue](https://github.com/qubic/archive-query-service/issues/140)).

### [Core Team](https://github.com/orgs/qubic/projects/1/views/5)

#### Core

* Add retry mechanisms for doge oracle queries that were not included in scheduled tick or failed due to e.g. timeout (see [PR](https://github.com/qubic/core/pull/810)).
* Harden doge share validation to verify the computor index (see [PR](https://github.com/qubic/core/pull/828)).
* Add check for doge oracle replies to decide whether they should be counted for revenue (active task, no duplicate) (see [PR](https://github.com/qubic/core/pull/828)).
* Revenue calculation v2 with doge factor (see [PR](https://github.com/qubic/core/pull/829)).
* Remove xmr custom mining(see [PR](https://github.com/qubic/core/pull/844)).
* Renable execution fees for digest computation (see [PR](https://github.com/qubic/core/pull/834)).
* Fix of three memory bugs (see [PR](https://github.com/qubic/core/pull/832)).
* Fix network due to stuck tick (see [PR](https://github.com/qubic/core/pull/831)).
* Addition of doubly-linked list to qpi (see [PR](https://github.com/qubic/core/pull/806)).
* Hotfix of tx status addon (see [PR](https://github.com/qubic/core/pull/848)).
* Add two flags to reduce false-positives in solution dedup (see [PR](https://github.com/qubic/core/pull/843)).
* QUTIL: add func for getting balances of 16 entities (see [PR](https://github.com/qubic/core/pull/835)).

#### Contract Verification

* Update contract-verify for oracle-related code (see [core PR](https://github.com/qubic/core/pull/839), [contract-verify PR](https://github.com/Franziska-Mueller/qubic-contract-verify/pull/5)).

#### Oracle Machine

* Update doge oracle service to (see [PR](https://github.com/qubic/oracle-machine/pull/11)).

#### SC Developer Support

* Review of QRWA update.
* Review ESCROW.
* Review SolanaBridge.
* Review QAgent.
* Review QUTIL update.

## Iteration 33

Duration: March 25, 2026 - April 7, 2026

### [Integration Team](https://github.com/orgs/qubic/projects/21)

* Go live with event logs endpoint (see [issue](https://github.com/qubic/qubic-deployment/issues/116), [issue](https://github.com/qubic/qubic-deployment/issues/109),
  [issue](https://github.com/qubic/qubic-deployment/issues/108), [issue](https://github.com/qubic/qubic-deployment/issues/118),
  and [issue](https://github.com/qubic/qubic-deployment/issues/110)).
* Add remaining filters to log events endpoint (see [issue](https://github.com/qubic/archive-query-service/issues/130)).
* Refactor the endpoint name getEvents to getEventLogs (see [issue](https://github.com/qubic/archive-query-service/issues/139)).

### [Core Team](https://github.com/orgs/qubic/projects/1/views/5)

#### Core

* Launched doge mining on mainnet (dispatcher, task and solution propagation, share validation via oracle) (see [PR](https://github.com/qubic/core/pull/825)).
* Speedup addition score (see [PR](https://github.com/qubic/core/pull/812))
* Fix compiler-dependent behavior with input qpi bit outside range 0/1 (see [PR](https://github.com/qubic/core/pull/822)).
* Implement special handling of DogeShareValidation user queries (see [PR](https://github.com/qubic/core/pull/811)).
* Add oracle interface for doge share validation (see [PR](https://github.com/qubic/core/pull/802)).

#### Oracle Machine

* Add doge share validation service (see [PR](https://github.com/qubic/oracle-machine/pull/10)).


## Iteration 32

Duration: March 11, 2026 - March 24, 2026

### [Core Team](https://github.com/orgs/qubic/projects/1/views/5)

#### Core

* Fix bug regarding misalignments after seamless epoch change (see [issue](https://github.com/qubic/core/issues/792))
* Work on minimum fee for SC procedures (see [issue](https://github.com/qubic/core/issues/653))
* Fix bug leading to changed order in asset iteration after snapshot loading (see [PR](https://github.com/qubic/core/pull/795))
* Write documentation for oracles (see [issue](https://github.com/qubic/core/issues/789), [PR](https://github.com/qubic/core/pull/796))
* Work on adding qubic scoring of doge mining via oracle machine (see [PR](https://github.com/qubic/core/issues/753)).
* Revenue formula v2, real data analysis (see [issue](https://github.com/qubic/core/issues/726)).
* Finish main net doge dispatcher implementation (see [issue](https://github.com/qubic/core/issues/750), code available in [repo](https://github.com/qubic/doge-connect)).
* Computor documentation for doge dispatcher (see [issue](https://github.com/qubic/core/issues/751), doc available in [repo](https://github.com/qubic/doge-connect)).

#### SC Developer Support
* Review and merge of QSurv.
* Review and merge of VottunBridge.
* Review and merge of Random.
* Review and merge of QVaultV2.
* Review Qusino.

### [Integration Team](https://github.com/orgs/qubic/projects/21)

* Store the last processed event log tick (see [issue](https://github.com/qubic/go-log-data-publisher/issues/25)).
* Add the last processed event log tick to status service (see [issue](https://github.com/qubic/go-data-publisher/issues/78)).
* Add the last processed event log tick to API (see [issue](https://github.com/qubic/archive-query-service/issues/124)).
* Improve archiver v2 performance with parallel calls (see [issue](https://github.com/qubic/go-archiver-v2/issues/19)).
* Support event log types 11 + 12 (see [issue](https://github.com/qubic/qubic-deployment/issues/111), [issue](https://github.com/qubic/archive-query-service/issues/134), and [issue](https://github.com/qubic/go-log-data-publisher/issues/28)).
* Support new event log filters (see [issue](https://github.com/qubic/archive-query-service/issues/117), [issue](https://github.com/qubic/archive-query-service/issues/125), and [issue](https://github.com/qubic/archive-query-service/issues/129)).
* Rename fields in the elastic log events index template (see [issue](https://github.com/qubic/qubic-deployment/issues/107)).
* Deploy another bob and bridge in dev (see [issue](https://github.com/qubic/qubic-deployment/issues/112)).
* Update duplicate check with the option to skip logs (see [issue](https://github.com/qubic/kafka-streams-services/issues/3)).
* Replace cl01 elastic cluster (see [issue](https://github.com/qubic/qubic-deployment/issues/114), and [issue](https://github.com/qubic/qubic-deployment/issues/115)).
* Remove ex03 volume (see [issue](https://github.com/qubic/qubic-deployment/issues/113)).

## Iteration 31

Duration: February 24, 2026 - March 10, 2026

### [Core Team](https://github.com/orgs/qubic/projects/1/views/5)

#### Core

* Implement oracle subscription support (see [issue](https://github.com/qubic/core/issues/673), [PR](https://github.com/qubic/core/pull/787)).
* Implement automatic state chagne detection for Smart Contracts (see [issue](https://github.com/qubic/core/issues/696), [PR](https://github.com/qubic/core/pull/774)).
* Start implementation core based minimum fee for SC procedures (see [issue](https://github.com/qubic/core/issues/653)).
* Speedup and double the number of mutations for addition mining algorithm (see [issue](https://github.com/qubic/core/issues/790) [PR](https://github.com/qubic/core/pull/785)).
* Revenue formula v2, shadow mode integration (see [issue](https://github.com/qubic/core/issues/726) [PR](https://github.com/qubic/core/pull/788)).
* First version of doge dispatcher finished, including local sandbox test with test dispatcher and test miner.

#### CLI

* Add oracle subscription commands for mainnet release in qubic-cli (see [PR](https://github.com/qubic/qubic-cli/pull/116)).
* Extend qlogging with subscriber log message (see [PR](https://github.com/qubic/qlogging/pull/11)).

#### Contract Verification

* Add new check for automatic state change detection (see [PR](https://github.com/Franziska-Mueller/qubic-contract-verify/pull/3)).

#### SC Developer Support
* Review of QSurv.
* Review of SolanaBridge.
* Review of VottunBridge.


### [Integration Team](https://github.com/orgs/qubic/projects/21)

* API: Get the qubic balance for multiple addresses (see [issue](https://github.com/qubic/qubic-aggregation/issues/2)).
* API: Get IPO bids by identity (see [issue](https://github.com/qubic/qubic-aggregation/issues/1)).
* API: Get Events by Identity (see [issue](https://github.com/qubic/archive-query-service/issues/108)).
* Switch bridge kafka library to franz-go (see [issue](https://github.com/qubic/go-log-data-publisher/issues/22)).
* Bridge: address bob /ws/logs removed endpoint (see [issue](https://github.com/qubic/go-log-data-publisher/issues/17)).
* Sort event logs by tick number and index/log id (see [issue](https://github.com/qubic/archive-query-service/issues/115)).
* Add secondary sort order for elastic index (see [issue](https://github.com/qubic/qubic-deployment/issues/99)).
* Improve log events consumer (see [issue](https://github.com/qubic/go-log-data-publisher/issues/21) and [issue](https://github.com/qubic/go-log-data-publisher/issues/20)).
* Improve get transactions for identity query (see [issue](https://github.com/qubic/archive-query-service/issues/119)).
* Improve handling of non-retriable errors in kafka publishers (see [issue](https://github.com/qubic/go-data-publisher/issues/74)).
* Improve error logging on consumers (see [issue](https://github.com/qubic/go-data-publisher/issues/76)).
* Fix grpc max recv/send message size (see [issue](https://github.com/qubic/archive-query-service/issues/127)).
* Fix public peer discovery issue (see [issue](https://github.com/qubic/go-qubic-nodes/issues/25)).
* Investigate proxy network issues on proxy03 (see [issue](https://github.com/qubic/qubic-deployment/issues/100)).
* Update Grafana version (see [issue](https://github.com/qubic/qubic-deployment/issues/105)).

## Iteration 30

Duration: February 10, 2026 - February 23, 2026

### [Integration Team](https://github.com/orgs/qubic/projects/21)

* Add an endpoint for getting IPO bids (see [issue](https://github.com/qubic/qubic-http/issues/47)).
* Create a single endpoint to fetch event logs (BETA) (see [issue](https://github.com/qubic/archive-query-service/issues/104)).
* Adapt event service to core changes (oracles) (see [issue](https://github.com/qubic/go-events/issues/28)).
* Store and expose logDigest in bridge service (see [issue](https://github.com/qubic/go-log-data-publisher/issues/11)).
* Investigate missing assets related events (see [issue](https://github.com/qubic/go-log-data-publisher/issues/9)).
* Metrics for bob log bridge service (see [issue](https://github.com/qubic/go-log-data-publisher/issues/10)).
* Do not ingest burn events with zero amount value into elastic (see [issue](https://github.com/qubic/go-log-data-publisher/issues/14)).
* Consume and ingest log events into elastic (see [issue](https://github.com/qubic/go-log-data-publisher/issues/3)).
* Remove encoded messages from the transaction hash (see [issue](https://github.com/qubic/go-log-data-publisher/issues/7)).

### [Core Team](https://github.com/orgs/qubic/projects/1/views/5)

#### Core

* Allow snapshot loading if computor count changes (see [PR](https://github.com/qubic/core/pull/766)).
* Add function for checking if public key belongs to contract (see [PR](https://github.com/qubic/core/pull/765)).
* Implement share validation for doge mining (see [issue](https://github.com/qubic/core/issues/749)).
* Configure github build workflow to fail on warnings to prevent PRs from introducing warnings (see [failing test PR](https://github.com/qubic/core/pull/768)).

#### Operations

* Created contract review guide in QCT repo to enable rotation for SC reviews (see [contract review guide](https://github.com/qubic/qct/blob/main/operations/contract_review.md)).

## Iteration 29

Duration: January 27, 2026 - February 09, 2026

### [Integration Team](https://github.com/orgs/qubic/projects/21)

* Define elastic search index for event logs (see [issue](https://github.com/qubic/go-log-data-publisher/issues/2)).
* Publish log events to kafka (see [issue](https://github.com/qubic/go-log-data-publisher/issues/1)).
* Deduplicate log events (see [issue](https://github.com/qubic/go-log-data-publisher/issues/4)).
* Automate openapi updates for integration documentation (see [issue](https://github.com/qubic/integration/issues/88)).
* Automate openapi updates for docs.qubic.org (see [issue](https://github.com/qubic/integration/issues/87)).
* Add https certificate for alternative grafana url (see [issue](https://github.com/qubic/qubic-deployment/issues/41)).
* Shut down ex02 (see [issue](https://github.com/qubic/qubic-deployment/issues/97)).

### [Core Team](https://github.com/orgs/qubic/projects/1/views/5)

#### Core
* Dogecoin mining design finished.
* Implement oracle user query support (see [PR](https://github.com/qubic/core/pull/754)).
* Finish oracle one-time contract query support (see [PR](https://github.com/qubic/core/pull/708)).
* Finalize oracle machine support in the core for the mainnet test (see [PR](https://github.com/qubic/core/pull/756)).
* Save/Load misalignment debug for Oracle.
* Price oracle for OM and some improvement/debug for connection OM <-> qubic node (see [issue](https://github.com/qubic/core/issues/701) [PR](https://github.com/qubic/oracle-machine/pull/4)).
* State change detection design finished and started implementation.
* Core side min fee for SC procedures design.

#### Contract Verification
* Add check that all globals are `constexpr` (`const` is not properly initialized in UEFI) (see [issue](https://github.com/qubic/core/issues/747)).

#### CLI

* Finish oracle query commands for mainnet release in qubic-cli (see [PR](https://github.com/qubic/qubic-cli/pull/114)).
* Finish extension of qlogging for oracle mainnet release (see [PR](https://github.com/qubic/qlogging/pull/10).

#### Documentation
* Add information about contract execution fees to docs.qubic.org (see [PR](https://github.com/qubic/docs/pull/51)).

#### SC Developer Support
* Review of new SC Pulse.

## Iteration 28

Duration: January 13, 2026 - January 26, 2026

### [Integration Team](https://github.com/orgs/qubic/projects/21)

* GetTransactionsForTick: Add source, destination, inputType, and amount filters (see [issue](https://github.com/qubic/archive-query-service/issues/94)).
* GetTransactionsForIdentity: Allow filtering by a single tick value (see [issue](https://github.com/qubic/archive-query-service/issues/93)).
* Extend address filtering for GetTransactionsForIdentity (see [issue](https://github.com/qubic/archive-query-service/issues/95)).
* Store TargetTickVoteSignature changes (see [issue](https://github.com/qubic/go-archiver-v2/issues/17)).
* Spike: Use BOB nodes to retrieve event log data (see [issue](https://github.com/qubic/integration/issues/58)).
* Fix error creating the archiver status (see [issue](https://github.com/qubic/go-data-publisher/issues/69)).
* Ops: Automate epoch archivation (see [issue](https://github.com/qubic/qubic-deployment/issues/91)).
* Ops: Dev01 migration to Dev02 (see [issue](https://github.com/qubic/qubic-deployment/issues/77)).
* Ops: Migrate ex01 to remote deployment (see [issue](https://github.com/qubic/qubic-deployment/issues/86)).
* Ops: Migrate ex03 to remote deployment (see [issue](https://github.com/qubic/qubic-deployment/issues/85)).
* Ops: Setup proxy03 (see [issue](https://github.com/qubic/qubic-deployment/issues/80)).
* Ops: Setup ex04 (see [issue](https://github.com/qubic/qubic-deployment/issues/79)).
* Ops: Analyze archiver-v1 backend usage (see [issue](https://github.com/qubic/qubic-deployment/issues/87)).
* Ops: Elastic cl02 maintenance (see [issue](https://github.com/qubic/qubic-deployment/issues/89)).

### [Core Team](https://github.com/orgs/qubic/projects/1/views/5)

#### Core

* Initial design and brainstorming for Dogecoin mining.
* Integrate and optimize for Dec 2025 mining verification into core (see [PR](https://github.com/qubic/core/pull/716) [PR](https://github.com/qubic/core/pull/721))
* Improve connection between Oracle Machine and Qubic node with Mock Oracle (see [issue](https://github.com/qubic/core/issues/727) [PR](https://github.com/qubic/core/pull/725) [PR](https://github.com/qubic/oracle-machine/pull/2))
* Initial design and brainstorming for state change detection and core side minimum fee for SCs.
* Fix race condition in using scratchpad / reorgBuffer (see [PR](https://github.com/qubic/core/pull/724)).

#### Operations and Marketing

* Created release guide in QCT repo to enable rotation for release creation (see [release guide](https://github.com/qubic/qct/blob/main/operations/release_guide.md)).
* Livestream AMA about execution fees.
* Review new whitepaper.

#### SC Developer Support

* Review of new SCs QReservePool, QThirtyFour, and QDuel.

## Iteration 27

Duration: December 30, 2025 - January 12, 2026

### [Core Team](https://github.com/orgs/qubic/projects/1/views/5)

* Preserve IDs of re-qualifying computors across epoch transition (see [issue](https://github.com/qubic/core/issues/654)).
* Improve tx priority for new entities (using latest incoming transfer tick instead of latest outgoing) (see [PR](https://github.com/qubic/core/pull/688)).
* Consistently exclude memory allocation for locals from execution fees of contract procedures (see [PR](https://github.com/qubic/core/pull/706), [PR](https://github.com/qubic/core/pull/707)).
* Temporarily disable contract execution fees for state digest computation.
* Collect and address feedback for Dec 2025 mining algo, test cases generation.

### [Integration Team](https://github.com/orgs/qubic/projects/21)

* Move legacy archiver database to larger disk (see [issue](https://github.com/qubic/qubic-deployment/issues/76)).
* Automatic spectrum upload (see [issue](https://github.com/qubic/qubic-deployment/issues/71)).
* Change lag-gate job to use the last processed tick (see [issue](https://github.com/qubic/qubic-deployment/issues/58)).
* Check computors list signature via system info call (see [issue](https://github.com/qubic/go-archiver-v2/issues/14)).
* Reindex computors and tick intervals on cl02 (see [issue](https://github.com/qubic/qubic-deployment/issues/78)).
* Change openapi docs to openapi v3 (see [issue](https://github.com/qubic/qubic-http/issues/41) and [issue](https://github.com/qubic/archive-query-service/issues/86)).
* Technical Analysis & Evaluation for API Documentation Standard (see [issue](https://github.com/qubic/integration/issues/70)).
* Deploy the API docs under our own domain (see [issue](https://github.com/qubic/integration/issues/86)).
* Document epoch transition procedures (see [issue](https://github.com/qubic/qubic-deployment/issues/72)).
