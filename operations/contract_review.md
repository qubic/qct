# Smart Contract Review

This file describes the checks that need to be done for a smart contract review.
All checks are described in the contracts doc (https://github.com/qubic/core/blob/main/doc/contracts.md).
Feel free to refer contirbutors to the appropriate sections of this doc for details.

## Automated Workflow Checks

We have two github workflows that are run on every pull request:
- **EFI Build:** This will build the whole solution file, including the `Qubic` and `test` projects. The compilation parameters are set to treat warnings as errors.
  Contributors need to make sure that their code compiles without warnings. ATTENTION: This workflow does not run the tests yet. Currently, the tests need to be run locally by a member of the core or test team.
- **Contract Verify:** The contract verify tool will do most of the checks regarding our restricted C++ language subset and some additional rules.
  If this workflow is failing, the contributor should build the tool locally to fix the errors (as described in https://github.com/qubic/core/blob/main/doc/contracts.md#review-and-tests).
  There are rare cases where the third-party parser of the tool may get stuck. The culprit lines can usually be found by commenting out parts of the code.

## Manual Checks

In addition to the automated workflows, the following checks need to be performed on a smart contract pull request:
- The base of the pull request is the `develop` branch. If the base code is outdated, contibutors should merge the most recent `develop` version again.
- The PR description should clearly describe the SC functionality or link to the proposal with the description.
- The PR changes/adds exactly the following files (smart contract pull requests cannot change/add any other files without getting special permission from the core devs):
  - `src/contract_core/contract_def.h`: Adding the new smart contract into the core code.
  - `src/contracts/<SCName>.h`: The smart contract header file.
  - `test/contract_<SCName>.cpp`: The test file for the new contract containing extensive unit tests.
  - `src/Qubic.vcxproj` and `src/Qubic.vcxproj.filters`: Adding the SC header file to the `Qubic` project in the VS Solution.
  - `test/test.vcxproj` and `test/test.vcxproj.filters`: Adding the SC test file to the `test` project in the VS Solution.
- The changes comply with the style guide (see https://github.com/qubic/core/blob/main/doc/contributing.md#style-guidelines).
- `src/contract_core/contract_def.h` requires three changes:
  - Adding the contract index and including the header file.
  - Adding the contract description: The construction epoch needs to be set correctly (proposal in epoch `x`, IPO in epoch `x+1`, construction in epoch `x+2`). As the size, the maximum of `sizeof(SCName)` and `sizeof(IPO)` has to be specified.
  - Registering contract functions and procedures via `REGISTER_CONTRACT_FUNCTIONS_AND_PROCEDURES(SCName)`.
- Additional checks for the contract header file:
  - Whenever an SC uses a `Collection/HashMap/HashSet` state variable and calls `remove(ByKey)` on it, a call to `cleanup/cleanupIfNeeded/reset` needs to be present as well (at least somewhere in the SC).
  - `LOG` macros are only allowed in procedures, not in functions.
  - Helper functions, i.e. functions defined as normal C++ functions, not using our macros, need to be `private/protected`, they cannot be `public`.
  - `while` loops need to be guaranteed to terminate.
