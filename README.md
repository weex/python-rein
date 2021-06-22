## Python-Rein

[![Build Status](https://travis-ci.org/ReinProject/python-rein.svg?branch=master)](https://travis-ci.org/ReinProject/python-rein)
[![Coverage
Status](https://coveralls.io/repos/ReinProject/python-rein/badge.svg?branch=master&service=github)](https://coveralls.io/github/ReinProject/python-rein?branch=master)

Rein makes freelancing for Bitcoin easier than ever. The simple, web-based python-rein client helps create jobs and bids as well as multisig escrows that keep funds safe. You don't need to stay online for others to find, bid or deliver on jobs.

We achieve decentralization by following the internet at large: cheap hosting paid for with bitcoin enables users to store data across a number of servers that may or may not be connected to one another. Since not a lot of data or bandwidth are required, we can have redundancy, reliability and censorship-resistance at low cost. Think $5 in bitcoin per year to have your data on 10 independently-run servers.

Servers can be paid for their services but they do not necessarily need to communicate with one other, and while the client attempts to validate what it sees, this validation is not complete nor has the software seen a security review. As such, you are advised to limit use of Rein to small jobs where loss of funds or time spent would not present a significant burden.

To read more about the project, visit http://reinproject.org.

## Development mailing list

https://groups.google.com/forum/#!forum/rein-development

## Getting started

Please see the [setup guide](doc/HOWTO-setup-rein.md). In a nutshell, run `rein start` to be guided through a web-based setup process.

## Upgrade

If you have 0.2 already installed and setup, please see our [note on upgrading](doc/Upgrading-v0.2-to-v0.3.md).

## Installation

Production:

    python setup.py install

Development:

    pip install --editable .

With the --editable switch, your edits immediately become live in your environment.

### Mediators

Each transaction in Rein requires a job creator, worker, and mediator. Most of the time it is expected that the mediator will do nothing but collect a small fee. In case of dispute, however, the mediator holds a third key refund, split or award escrowed funds using their best judgement.

### Job creators and workers

Run the following command to get started.

    rein start

This will guide you through a setup process and provide a simple interface for you to use Rein. 

### The rest of the order flow

After a job posting and bids have been made, the job creator will use _offer_ to choose a bid. They should then fund both the primary and mediator escrow addresses so the worker can begin. Once the work is complete, the worker uses _deliver_ to post deliverables which can be reviewed and accepted by the job creator with _accept_. During the _accept_ step, the job creator will be prompted for signed primary and mediator payment transactions. 

Python-rein currently cannot query for unspent outputs or assemble transactions so it is recommended that you download a copy of [ReinProject's fork of Coinb.in](https://github.com/ReinProject/coinbin) in order to sign (and for the worker and mediator, broadcast) the payment transactions.

If you want detailed information about pending jobs, click on the job in question after running `rein start`. 

## Addresses and Payments

### Address generation

Rein generates your Bitcoin addresses automatically at setup. It uses BIP32/39 to first generate a secret phrase, then transform it into a main private key and derive child keys from it.

Here is the derivation path:
m/0 - master address, private, and public keys
m/1'/0 - delegate address, private, and public keys

The mnemonic should be kept private as it is the key to accessing your Rein account and signing escrow release.

-----

Rein uses two types of escrow in order to protect funds that are to go to the worker or job creator (depending upon the success of the transaction) as well as the mediator. 

### Primary payment

For the primary payment, a simple 2-of-3 escrow address is created. To spend the funds placed at this address requires that two parties out of the job creator, worker, and mediator sign each payment. The user is prompted for a signed primary payment if they are a job creator accepting a delivery or they are the mediator resolving a dispute. This payment can be built and signed using Coinb.in which will retrieve unspent outputs (i.e. the funds) and allow the user to specify the destination address and amount, with excess going to the Bitcoin network as a fee for the miner of the transaction. 

Having only been signed by one party, this payment should be reviewed before the second party adds their signature and broadcasts the transaction.

For example in a normal, non-disputed transaction the worker should check that their address is the one being paid and that the Bitcoin network fee is reasonable. If an error is found, the signing party can be contacted to build a new transaction with the correct information.

### Mediator payment

For the mediator payment, a mandatory multisig address is created. To spend the funds placed at this address, the mediator must sign the payment and be accompanied by either the job creator or worker's signature. This ensures that the job creator and worker cannot conspire to steal the mediator payment, even though in theory they could both refuse to sign the mediator's payment if both feel the mediator made an unfair judgement.

Like with the primary payment, a user is prompted for a signed mediator payment if they are a job creator accepting a delivery or they are the mediator resolving a dispute.

## Developer Notes

We have [a quick Vagrant-based setup](https://github.com/ReinProject/devsetup) that gives you a virtual machine with the python-rein client, Causway server and its Bitcoin Core (testnet) node all configured to work together. Testing usually involves creating users and walking through jobs so a virtual machine that has all components going, even allowing payments to be sent is very helpful.

To generate or update pot files for translation, run the following from the root of the repo:

    xgettext.pl rein/cli.py rein/lib/*.py

## Testing

We have [a quick Vagrant-based setup](https://github.com/ReinProject/devsetup) that gives you a virtual machine with the python-rein client, Causway server and its Bitcoin Core (testnet) node all configured to work together. Testing usually involves creating users and walking through jobs so a virtual machine that has all components going, even allowing payments to be sent is very helpful.

Tests are run using nose, with the make file specifying two different test commands:

    $ make test
    nosetests -v --ignore-files="test_cli.py"
    ..
    ----------------------------------------------------------------------
    Ran 5 tests in 0.344

    OK

and

    $ make test_all
    nosetests -v
    ..
    ----------------------------------------------------------------------
    Ran 7 tests in 2.393s

    OK

By default, the more limited "test" command should be used when a dedicated test-environment, such as the Vagrant set-up mentioned above, is not available. It currently runs all tests except the tests specified in "tests/test_cli.py".

Tox fails right now but does run flake so will be helpful for cleanup.

Be aware that new unit tests should be added to a file within the tests directory. Both the file name and the test method names should follow the naming convention "test_*".
