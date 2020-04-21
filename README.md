# rosetta-validator

[![Coinbase](https://circleci.com/gh/coinbase/rosetta-validator/tree/master.svg?style=shield)](https://circleci.com/gh/coinbase/rosetta-validator/tree/master)
[![Coverage Status](https://coveralls.io/repos/github/coinbase/rosetta-validator/badge.svg)](https://coveralls.io/github/coinbase/rosetta-validator)
[![Go Report Card](https://goreportcard.com/badge/github.com/coinbase/rosetta-validator)](https://goreportcard.com/report/github.com/coinbase/rosetta-validator)
[![License](https://img.shields.io/github/license/coinbase/rosetta-validator.svg)](https://github.com/coinbase/rosetta-validator/blob/master/LICENSE.txt)

Once you create a Rosetta server, you'll need to test its
performance and correctness. This validation tool makes that easy!

## What is Rosetta?
Rosetta is a new project from Coinbase to standardize the process
of deploying and interacting with blockchains. With an explicit
specification to adhere to, all parties involved in blockchain
development can spend less time figuring out how to integrate
with each other and more time working on the novel advances that
will push the blockchain ecosystem forward. In practice, this means
that any blockchain project that implements the requirements outlined
in this specification will enable exchanges, block explorers,
and wallets to integrate with much less communication overhead
and network-specific work.

## Install
```
go get github.com/coinbase/rosetta-validator
```

## Usage
```
A simple CLI to validate a Rosetta server

Usage:
  rosetta-validator [command]

Available Commands:
  check:complete Run a full check of the correctness of a Rosetta server
  check:quick    Run a simple check of the correctness of a Rosetta server
  help           Help about any commands

Global Flags:
  --account-concurrency uint       concurrency to use while fetching accounts during reconciliation (default 8)
  --block-concurrency uint         concurrency to use while fetching blocks (default 8)
  --data-dir string                folder used to store logs and any data used to perform validation (default "./validator-data")
  --end int                        block index to stop syncing (default -1)
  --halt-on-reconciliation-error   Determines if block processing should halt on a reconciliation
                                   error. It can be beneficial to collect all reconciliation errors or silence
                                   reconciliation errors during development. (default true)
  --log-balance-changes            log balance changes (default true)
  --log-blocks                     log processed blocks (default true)
  --log-reconciliations            log balance reconciliations (default true)
  --log-transactions               log processed transactions (default true)
  --server-url string              base URL for a Rosetta server to validate (default "http://localhost:8080")
  --start int                      block index to start syncing (default -1)
  --transaction-concurrency uint   concurrency to use while fetching transactions (if required) (default 16)
```

### check:complete
```
Check all server responses are properly constructed, that
there are no duplicate blocks and transactions, that blocks can be processed
from genesis to the current block (re-orgs handled automatically), and that
computed balance changes are equal to balance changes reported by the node.

When re-running this command, it will start where it left off. If you want
to discard some number of blocks populate the --start flag with some block
index less than the last computed block index.

Usage:
  rosetta-validator check:complete [flags]

Flags:
      --bootstrap-balances string   Absolute path to a file used to bootstrap balances before starting syncing.
                                    Populating this value after beginning syncing will return an error.
  -h, --help                        help for check:complete
      --lookup-balance-by-block     When set to true, balances are looked up at the block where a balance
                                    change occurred instead of at the current block. Blockchains that do not support
                                    historical balance lookup should set this to false. (default true)
```

### check:quick
```
Check all server responses are properly constructed and that
computed balance changes are equal to balance changes reported by the
node. To use check:quick, your server must implement the balance lookup
by block.

Unlike check:complete, which requires syncing all blocks up
to the blocks you want to check, check:quick allows you to validate
an arbitrary range of blocks (even if earlier blocks weren't synced).
To do this, all you need to do is provide a --start flag and optionally
an --end flag.

It is important to note that check:quick does not support re-orgs and it
does not check for duplicate blocks and transactions. For these features,
please use check:complete.

When re-running this command, it will start off from genesis unless you
provide a populated --start flag. If you want to run a stateful validation,
use the check:complete command.

Usage:
  rosetta-validator check:quick [flags]

Flags:
  -h, --help   help for check:quick

```

## Development
* `make deps` to install dependencies
* `make test` to run tests
* `make lint` to lint the source code (included generated code)
* `make release` to run one last check before opening a PR

## Correctness Checks
This tool performs a variety of correctness checks using the Rosetta Server. If
any correctness check fails, the validator will exit and print out a detailed
message explaining the error.

### Response Correctness
The validator uses the autogenerated [Go Client package](https://github.com/coinbase/rosetta-sdk-go)
to communicate with the Rosetta Server and assert that responses adhere
to the Rosetta interface specification.

### Duplicate Hashes
The validator checks that a block hash or transaction hash is
never duplicated.

### Non-negative Balances
The validator checks that an account balance does not go
negative from any operations.

### Balance Reconciliation
#### Active Addresses
The validator checks that the balance of an account computed by
its operations is equal to the balance of the account according
to the node. If this balance is not identical, the validator will
exit.

#### Inactive Addresses
The validator randomly checks the balances of accounts that aren't
involved in any transactions. The balances of accounts could change
on the blockchain node without being included in an operation
returned by the Rosetta Server. Recall that **ALL** balance-changing
operations must be returned by the Rosetta Server.

## Future Work
* Move syncer, reconciler, and storage packages to rosetta-sdk-go for better re-use.
* Automatically test the correctness of a Rosetta Client SDK by constructing,
signing, and submitting a transaction. This can be further extended by ensuring
broadcast transactions eventually land in a block.
* Change logging to utilize a more advanced output mechanism than CSV.

## License
This project is available open source under the terms of the [Apache 2.0 License](https://opensource.org/licenses/Apache-2.0).

© 2020 Coinbase