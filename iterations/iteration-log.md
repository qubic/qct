# Iteration Results

This page provides a simplified list of iteration deliverables. It is not intended to be exhaustive,
please see the respective team boards and repositories on the [qubic GitHub](https://github.com/qubic) for that.
Listed items are completed but that does not mean that all the deliverables are deployed to production yet.

## Iteration 2 

Duration: Jan 08 2025 - Jan 27 2025

### [Integration Team](https://github.com/orgs/qubic/projects/21)

* Implemented archiver p2p synchronization protocol (see [issue](https://github.com/qubic/go-archiver/issues/43)).
* Add paging to the transfers endpoint (see [issue](https://github.com/qubic/go-archiver/issues/95)).
* Improved retry mechanism for node communication (see [issue](https://github.com/qubic/qubic-http/issues/29)).
* Improved architecture documentation (see [issue](https://github.com/qubic/qubic-deployment/issues/6) and [issue](https://github.com/qubic/integration/issues/30)).
* Improved GetEmptyTicksForEpochs performance (see [issue](https://github.com/qubic/go-archiver/issues/90)).
* Provide service for asset owners list (see  [issue](https://github.com/qubic/qx-service/issues/6) and [assets-service](https://github.com/qubic/assets-service)).

#### QX (not part of integration infrastructure)

* Aggregating order book entries (see [issue](https://github.com/qubic/qx-service/issues/8)).
* Help creating qx orders (see [issue](https://github.com/qubic/qx-service/issues/7)).

### [Core Team](https://github.com/orgs/qubic/projects/1/views/8)

#### Core

* Added first test for custom mining tasks: nodes can receive custom mining solutions (see [PR](https://github.com/qubic/core/pull/271)).
* Improved coding style consistency for class names (see [PR](https://github.com/qubic/core/pull/272)).
* Added pause before clearing all memory at epoch transition.
* Combined memory allocation and error logging in one function (see [issue](https://github.com/qubic/core/issues/215)).
* Added transfer of asset management rights (see [issue](https://github.com/qubic/core/issues/269)).
* Fixed failing testcases when logging is enabled (see [issue](https://github.com/qubic/core/issues/281)).
* Evaluated possibilities to measure network speed by airdrop (see [issue](https://github.com/qubic/core/issues/274)).

#### CLI

* Added compression and decompression for upload and download of files (see [issue](https://github.com/qubic/core/issues/251)).
* Improved precision of command nonce to enable execution of commands in fast succession (see [issue](https://github.com/qubic/qubic-cli/issues/57)).
* Fixed Issue with receiving information from noded that have no index yet (see [PR](https://github.com/qubic/qubic-cli/pull/55))
#### SC Developer Support

* Guided QEarn developers in adding variables for metrics to QEarn state and reviewed their changes in core, CLI, and the standalone state migration tool. 
