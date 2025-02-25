# Iteration Results

This page provides a simplified list of iteration deliverables. It is not intended to be exhaustive,
please see the respective team boards and repositories on the [qubic GitHub](https://github.com/qubic) for that.
Listed items are completed but that does not mean that all the deliverables are deployed to production yet.

## Iteration 4

Duration: Feb 11, 2025 - Feb 24, 2025

### [Integration Team](https://github.com/orgs/qubic/projects/21)

* Improve node pooling: minimum tick (see [issue](https://github.com/qubic/go-qubic-nodes/issues/13)).
* Improve node pooling: node selection by highest tick (see [issue](https://github.com/qubic/go-qubic-nodes/issues/11)).
* Add code and documentation for transferring assets (see [issue](https://github.com/qubic/go-node-connector/issues/14)).
* Make arbitrator key configurable to be able to run archive service on testnet (see [issue](https://github.com/qubic/go-archiver/issues/100)).
* Add metrics to assets service (see [issue](https://github.com/qubic/assets-service/issues/2)).
* Add metrics to transfers service (see [issue](https://github.com/qubic/go-transfers/issues/4)).
* Add metrics to events service (see [issue](https://github.com/qubic/go-events/issues/10)).
* Update deployment information for dev02 (see [issue](https://github.com/qubic/qubic-deployment/issues/10)).

### [Core Team](https://github.com/orgs/qubic/projects/1/views/8)

#### Core

* Added feature to query asset information with various filtering options (see [issue](https://github.com/qubic/core/issues/266), [PR](https://github.com/qubic/core/pull/313)).
* Added first version of coding guidelines and updated documentation (see [PR](https://github.com/qubic/core/pull/319)).
* Added procedures for QPI functions in Test Example A SC (to enable automatic testing in testnet in the next step) (see [issue](https://github.com/qubic/core/issues/291), [PR](https://github.com/qubic/core/pull/320)).
* Added some network message documentation, did some refactoring, and changed `TryAgain` type (see [PR](https://github.com/qubic/core/pull/314)).
* Added dispatcher in preparation of custom mining (see [PR](https://github.com/qubic/core/pull/310)).
* Fixed issue of test.exe not starting anymore (see [PR](https://github.com/qubic/core/pull/308)).

#### CLI

* Implemented `-queryasset` (see [PR](https://github.com/qubic/qubic-cli/pull/70)).


## Iteration 3

Duration: Jan 28, 2025 - Feb 10, 2025

### [Core Team](https://github.com/orgs/qubic/projects/1/views/8)

#### Core

* Enabled contracts to bid for shares during a contract IPO via QPI function (see [issue](https://github.com/qubic/core/issues/293)).
* Supported more QPI functions in Google Test environment (see [issue](https://github.com/qubic/core/issues/294)).
* Solved deadlock in asset management rights transfer (see [issue](https://github.com/qubic/core/issues/296)).
* Worked on various improvements to ticking speed and saftey (see [issue](https://github.com/qubic/core/issues/305)).
* General info about Monero mining (see [issue](https://github.com/qubic/core/issues/275)).
* Implemented qutil SC procedure for benchmarking transfer speed/throughput (see [PR](https://github.com/qubic/core/pull/300)).
* Manually tested the docker for running a Qubic test node (see [issue](https://github.com/qubic/core/issues/284)).

#### CLI

* Revised how socket's recv() is used, fixing that occasionally incomplete data was received, for example about quorum ticks (see [issue](https://github.com/qubic/core/issues/304)).
* Fixed the offset between local and node time in the synctime command (see [PR](https://github.com/qubic/qubic-cli/pull/67)).
* Implemented `-qutilsendtomanybenchmark` (see [PR](https://github.com/qubic/qubic-cli/pull/69)).

#### SC Developer Support

* Final review of MSVAULT contract
* Answering questions of contract devs in private messages

### [Integration Team](https://github.com/orgs/qubic/projects/21)

* Researched and created prototype for long term archive solution (base for next steps).
* Improved monitoring of go-qubic-nodes service (see [issue](https://github.com/qubic/go-qubic-nodes/issues/5)).
* Added possibility to use pass codes per qubic node (see [issue](https://github.com/qubic/go-qubic/issues/8)).
* Used pass code per qubic node in go-events service (see [issue](https://github.com/qubic/go-events/issues/7)).
* Refactored code for validating requested tick range in go-archiver (see [issue](https://github.com/qubic/go-archiver/issues/92)).

## Iteration 2

Duration: Jan 08, 2025 - Jan 27, 2025

### [Integration Team](https://github.com/orgs/qubic/projects/21)

* Implemented archiver p2p synchronization protocol (see [issue](https://github.com/qubic/go-archiver/issues/43)).
* Add paging to the transfers endpoint (see [issue](https://github.com/qubic/go-archiver/issues/95)).
* Improved retry mechanism for node communication (see [issue](https://github.com/qubic/qubic-http/issues/29)).
* Improved architecture documentation (see [issue](https://github.com/qubic/qubic-deployment/issues/6) and [issue](https://github.com/qubic/integration/issues/30)).
* Improved GetEmptyTicksForEpochs performance (see [issue](https://github.com/qubic/go-archiver/issues/90)).
* Provide service for asset owners list (see [issue](https://github.com/qubic/qx-service/issues/6) and [assets-service](https://github.com/qubic/assets-service)).

#### QX (not part of integration infrastructure)

* Aggregating order book entries (see [issue](https://github.com/qubic/qx-service/issues/8)).
* Help creating qx orders (see [issue](https://github.com/qubic/qx-service/issues/7)).

### [Core Team](https://github.com/orgs/qubic/projects/1/views/8)

#### Core

* Added first test for custom mining tasks: nodes can receive custom mining solutions (see [PR](https://github.com/qubic/core/pull/271)).
* Improved coding style consistency for class names (see [PR](https://github.com/qubic/core/pull/272)).
* Added pause before clearing all logging memory at epoch transition for facilitating complete event archive (multiple single commits including some fixes).
* Combined memory allocation and error logging in one function (see [issue](https://github.com/qubic/core/issues/215)).
* Added transfer of asset management rights and fixed a few bugs (see [issue](https://github.com/qubic/core/issues/269)).
* Fixed failing testcases when logging is enabled (see [issue](https://github.com/qubic/core/issues/281)).
* Evaluated possibilities to measure network speed by airdrop (see [issue](https://github.com/qubic/core/issues/274)).
* Added async file I/O support (see [PR](https://github.com/qubic/core/pull/253)).

#### CLI

* Added compression and decompression for upload and download of files (see [issue](https://github.com/qubic/core/issues/251)).
* Improved precision of command nonce to enable execution of commands in fast succession (see [issue](https://github.com/qubic/qubic-cli/issues/57)).
* Fixed issue with receiving information from nodes that have no index yet (see [PR](https://github.com/qubic/qubic-cli/pull/55)).
* Added command to transfer asset management rights from QX to another contract (see [PR](https://github.com/qubic/qubic-cli/pull/64)).
* Fixed issue in getting tick transactions (see commits [1](https://github.com/qubic/qubic-cli/commit/02fc287cac5d3df8a317f019bccc8dddd6432396) and [2](https://github.com/qubic/qubic-cli/commit/bfb104a63eb27dfe34c85bf67c50055ed262dc9a))

#### SC Developer Support

* Guided QEarn developers in adding variables for metrics to QEarn state and reviewed their changes in core, CLI, and the standalone state migration tool.
* Supporting QVAULT developers within several issues.
