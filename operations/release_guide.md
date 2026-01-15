# Release Guide
This guide is the authoritative playbook for safely releasing Qubic core updates across epoch boundaries. It exists to prevent consensus errors, coordinate developers and operators, and ensure that every network upgrade is predictable, testable, and reversible when necessary.

## Step-by-Step Release Guide

- On Monday, check which PRs are approved and ready to be merged. Ask proactively in the Core Dev Chat if there are more changes planned.
- Any changes that impact downstream clients (for example modifying external interfaces, communication protocol, existing message structs, logging events, etc.) need to be communicated to integration/ecosystem developers at least 1 week ahead, usually via the `#dev` channel on discord.
- When merging PRs into `develop`, it is usually the best option to do the "squash and merge" version which will bundle all changes in a single new commit. This allows for easy reverting if issues are found during the release test. Only use the "create merge commit" version if there is significant merit in keeping all original commits. Note that this will make a clean revert of a single feature much harder. We have seen unexpected results with trying to revert "merge commits" in the past. 
- Some changes (e.g. new SCs) might depend on the outcome of a proposal. In this case, merge the PR and afterwards add a preprocessor toggle to easily let the computors disable the change (see [example commit](https://github.com/qubic/core/commit/e9293e7cce0d7f18e7db87aa11d762260deb72e7)).
- Decide if the transition will be seamless or not. Every change that affects consensus/digests has to be done non-seamless. Continue with the appropriate subsection below.

## Non-Seamless Epoch Transition (with Breaking Changes)
- Use [this Google Sheet](https://docs.google.com/spreadsheets/d/1y52UHzO3vtLkzHyJVr2F_1zQ8PTsAUwDOORwpxk3Mww/edit?usp=sharing) to estimate the new initial tick (to minimize the DDoS attack surface, the new initial tick should fall into the external mining window (`tick % (676+677) > 676`). The sheet will mark this in red if it’s not the case.
- In `public_settings.h`, adjust the epoch, version, and initial tick. We use semantic version names `x.y.z` where versions with the same `x` and `y` should be able to run together in a network. Hence, when doing non-seamless, the new version will always be `x.(y+1).0`. `START_NETWORK_FROM_SCRATCH` should be set to 1 for non-seamless.
- Create a testnet branch (called `testnets/release-...`) by branching from `develop` and cherry-picking the testnet config commit (latest commit in the `testnet` branch in the core repo).
- Ping the testing team in the `#core-testing` channel to start the release test. The general release test protocol can be found [here](https://github.com/qubic/core/blob/main/test/README.md). If new features require additional tests, specify them explicitly. The release test should be started ideally on Monday or latest Tuesday morning UTC to allow enough time.
- Once the test is successful, create a pull request from `develop` to `main`. Write the change log in the PR description. Ping in the Core Dev Chat to get a reviewer for the release PR.
- Double check that the new initial tick that was estimated still works, i.e. the network will not reach it before epoch transition. You can do this by updating the “current tick” and “current time” in the sheet used before.
- As soon as the release PR is approved, merge the PR into `main` (using the “create merge commit” version to preserve the commits from `develop`).
- Wait for the automatic workflows to successfully pass.
- Create a release in the core repo. Please make sure to match the formatting of previous releases. The release description should contain the change log (copy from release PR).
- Make an announcement in the `#computor-operator` channel. See [this Google Doc](https://docs.google.com/document/d/1OPSJv3624oKI36rLv5vqLeKxv6vJ-fz2c1l6PHWb560/edit?usp=sharing) for examples for different scenarios. The announcement should also contain the change log (copy from release/PR).

## Seamless Epoch Transition (with Non-Breaking Changes)
- For seamless transition, the optional update will only be released after epoch transition. However, you need to test that the optional update indeed runs together with the previous version.
- Collect all non-breaking changes in the `develop` branch.
- In `public_settings.h`, adjust the epoch and version (initial tick has to be set after epoch transition). We use semantic version names `x.y.z` where versions with the same `x` and `y` should be able to run together in a network. Hence, when doing seamless, the new version will always be `x.y.(z+1)`. `START_NETWORK_FROM_SCRATCH` should be set to 0 for seamless.
- Create a testnet branch (called `testnets/release-...`) by branching from `develop` and cherry-picking the testnet config commit (latest commit in the `testnet` branch in the core repo).
- Ping the testing team in the `#core-testing` channel to start the release test. Make sure to describe the steps to test the optional update, mimicking what happens in the real network (all steps will be performed by the testing team):
    - (1) start a network of at least two nodes with the current epoch’s release, 
    - (2) wait for the nodes to do a seamless epoch transition,
    - (3) in the code for the optional update, adapt the initial tick in `public_settings.h` to match the correct tick for the new epoch,
    - (4) restart a subset of the nodes with the optional update,
    - (5) make sure the network runs stable with both versions running together. 

    The general release test protocol can be found [here](https://github.com/qubic/core/blob/main/test/README.md). If new features require additional tests, specify them explicitly. The release test should be started ideally on Monday or latest Tuesday morning UTC to allow enough time.
- Once the test is successful, create a pull request from `develop` to `main`. Write the change log in the PR description. Ping in the Core Dev Chat to get a reviewer for the release PR. **Wait for epoch transition before merging!**
- Wait for seamless epoch transition and query the new initial tick from a node (e.g. using qubic-cli). Update the initial tick in `public_settings.h` in the `develop` branch.
- Once the PR is approved and the initial tick is updated, merge the PR into `main` (using the “create merge commit” version to preserve the commits from `develop`).
- Wait for the automatic workflows to successfully pass.
- Create a release in the core repo. Please make sure to match the formatting of previous releases. The release description should contain the change log (copy from release PR).
- Make an announcement in the `#computor-operator` channel. See [this Google Doc](https://docs.google.com/document/d/1OPSJv3624oKI36rLv5vqLeKxv6vJ-fz2c1l6PHWb560/edit?usp=sharing) for examples for different scenarios. The announcement should also contain the change log (copy from release/PR).

## Seamless Epoch Transition (without Changes/Restart Only)
- Wait for seamless epoch transition and query the new initial tick from a node (e.g. using qubic-cli). 
- [ can be done directly in the `main` branch ] In `public_settings.h`, adjust the epoch, version, and initial tick. We use semantic version names `x.y.z` where versions with the same `x` and `y` should be able to run together in a network. Hence, when doing seamless, the new version will always be `x.y.(z+1)`. `START_NETWORK_FROM_SCRATCH` should be set to 0 for seamless.
- Wait for the automatic workflows to successfully pass.
- Create a release in the core repo. Please make sure to match the formatting of previous releases.
- Make an announcement in the `#computor-operator` channel. The message for the restart version can be very short, it only really needs to include the link to the release.

## After Epoch Transition
When the network runs stable, revert any temporary changes in develop for the next release, for example:
- Preprocessor toggles used for code that depended on a proposal.
- One-time code used by SCs to initialize new state variables after state size changes (usually found in `BEGIN_EPOCH` within an `if (epoch == x)` block). These may be copied to `INITIALIZE` for later reference. `INITIALIZE` will never be called again on a contract that is already constructed.
- Other one-time bug fixes (anything wrapped in an `if (epoch == x)` block is a good candidate).
