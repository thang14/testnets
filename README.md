# Connect to a Testnet

This document explains how to connect to the Testnet of a [Cosmos-SDK](https://github.com/cosmos/cosmos-sdk/) based blockchain. It can be used to connect to the latest Testnet for the Cosmos Hub.

## Software Setup (Manual Installation)

Follow these instructions to install the Cosmos-SDK and connect to the latest Testnet. This instructions work for both a local machine and a VM in a cloud server.

If you want to run a non-validator full-node, installing the SDK on a Cloud server is a good option. However, if you are want to become a validator for the Hub's `mainnet` you should look at more complex setups, including [Sentry Node Architecture](https://github.com/cosmos/cosmos/blob/master/VALIDATORS_FAQ.md#how-can-validators-protect-themselves-from-denial-of-service-attacks), to protect your node from DDOS and ensure high-availability (see the [technical requirements](https://github.com/cosmos/cosmos/blob/master/VALIDATORS_FAQ.md#technical-requirements)). You can find more information on validators in our [website](https://cosmos.network/validators), in the [Validator FAQ](https://cosmos.network/resources/validator-faq) and in the [Validator Chat](https://riot.im/app/#/room/#cosmos_validators:matrix.org).

### Install [Go](https://golang.org/)

Install `go` following the [instructions](https://golang.org/doc/install) in the official golang website.
You will require **Go 1.10+** for this tutorial.

#### Set GOPATH

First, you will need to set up your `GOPATH`. Make sure that the location `$HOME` is something like `/Users/<username>`, you can corroborate it by typing `echo $HOME` in your terminal.

Go to `$HOME` with the command `cd $HOME` and open the the hidden file `.bashrc` with a code editor and paste the following lines \(or `.bash_profile` if your're using OS X\).

```
export GOPATH=$HOME/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

Save and restart the terminal.

_Note_: If you can't see the hidden file, use the shortcut `Command + Shift + .` in Finder.


### Install [GNU Wget](https://www.gnu.org/software/wget/)

**MacOS**

```
brew install wget
```

**Linux**

```
sudo apt-get install wget
```

Note: You can check other available options for downloading `wget` [here](https://www.gnu.org/software/wget/faq.html#download).

### Install Gaia

Now we can fetch the correct versions of each dependency by running:

```
mkdir -p $GOPATH/src/github.com/cosmos/cosmos-sdk
git clone https://github.com/cosmos/cosmos-sdk.git
git checkout v0.18.0-rc0
make get_tools // run $ make update_tools if already installed
make get_vendor_deps
make install
```

This will install the `gaiad` and `gaiacli` binaries. Verify that everything is OK by running:

```
gaiad version
```

You should see:

```
0.18.0-dev-af15f89
```

And also:

```
gaiacli version
```

You should see:

```
0.18.0-dev-af15f89
```

## Full Node Setup

Clone the testnet repo.

```
git clone https://github.com/cosmos/testnets

mkdir -p $HOME/.gaiad/config
cp -a testnets/gaia-6000/genesis.json $HOME/.gaiad/config/genesis.json
gaiad unsafe_reset_all
```

Add a seed node by changing `seed = ""` in `config.toml` to `seed = "80a35a46ce09cfb31ee220c8141a25e73e0b239b@35.198.166.171:46656"`.

Lastly change the `moniker` string in the`config.toml`to identify your node.

```
# A custom human readable name for this node
moniker = "<your_custom_name>"
```

## Run a Full Node

Start the full node:

```
gaiad start
```

Check the everything is running smoothly:

```
gaiacli status
```

## Generate keys

You'll need a private and public key pair \(a.k.a. `sk, pk` respectively\) to be able to receive funds, send txs, bond tx, etc.

To generate your a new key \(default _ed25519_ elliptic curve\):

```
gaiacli keys add <your_key_name>
```

Next, you will have to enter a passphrase for your key twice. Save the _seed_ _phrase_ in a safe place in case you forget the password.

Now if you check your private keys you will see the `<your_key_name>` key among them:

```
gaiacli keys show <your_key_name>
```

You can see all your other available keys by typing:

```
gaiacli keys list
```

The validator pubkey from your node should be the same as the one printed with the command:

```
gaiad tendermint show_validator
```

Finally, save your address and pubkey into a variable to use them afterwards.

```
MYADDR=<your_newly_generated_address>
MYPUBKEY=<your_newly_generated_public_key>
```

**IMPORTANT:** We strongly recommend to **NOT** use the same passphrase for your different keys. The Tendermint team and the Interchain Foundation will not be responsible for the lost of funds.

### Get coins

The best way to get coins at the moment is to ask in Riot chat. We plan to have a reliable faucet in future testnets.

## Send tokens

```
gaiacli send --amount=1000fermion --chain-id=<name_of_testnet_chain> --sequence=0 --name=<key_name> --to=<destination_address>
```

The `--amount` flag defines the corresponding amount of the coin in the format `--amount=<value|coin_name>`

The `--sequence` flag corresponds to the sequence number to sign the tx.

Now check the destination account and your own account to check the updated balances \(by default the latest block\):

```
gaiacli account <destination_address>
gaiacli account <your_address>
```

You can also check your balance at a given block by using the `--block` flag:

```
gaiacli account <your_address> --block=<block_height>
```

## Run a Validator Node

[Validators](https://cosmos.network/validators) are actors from the network that are responsible from committing new blocks to the blockchain by submitting their votes. In terms of security, validators' stake is slashed in all the zones they belong if they become unavailable, double sign a transaction, or don't cast their votes. We strongly recommend entities intending to run validators in the Cosmos Hub's `mainnet` to check the [technical requirements](https://github.com/cosmos/cosmos/blob/master/VALIDATORS_FAQ.md#technical-requirements) and take the necessary precautions to ensure high-availability, such as setting a Sentry Node architecture. If you have any question about validators, read the [Validator FAQ](https://cosmos.network/resources/validator-faq) and join the [Validator Chat](https://riot.im/app/#/room/#cosmos_validators:matrix.org).

This section covers the instructions necessary to stake tokens to become a testnet validator candidate.

Your `pubkey` can be used to create a new validator candidate by staking some tokens:

You can find your node pubkey by running
```
gaiad show_validator
```

and this returns your public key for the declare-candidate command


```
gaiacli declare-candidacy --amount=500steak --pubkey=<your_node_pubkey> --address-candidate=<your_address> --moniker=satoshi --chain-id=<name_of_the_testnet_chain> --sequence=1 --name=<key_name>
```

You can add more information of the validator candidate such as`--website`, `--keybase-sig `or additional `--details`. If you want to edit the candidate info:

```
gaiacli edit-candidacy --details="To the cosmos !" --website="https://cosmos.network"
```

Finally, you can check all the candidate information by typing:

```
gaiacli candidate --address-candidate=<your_address> --chain-id=<name_of_the_testnet_chain>
```

To check that the validator is active you can find it on the validator set list:

```
gaiacli validatorset
```

**Note:** Remember that to be in the validator set you need to have more total power than the Xnd validator, where X is the assigned size for the validator set \(by default _`X = 100`_\).

## Delegate your tokens

You can delegate \(_i.e._ bind\) **Atoms** to a validator to become a [delegator](https://cosmos.network/resources/delegators) and obtain a part of its fee revenue in **Photons**. For more information about the Cosmos Token Model, refer to our [whitepaper](https://github.com/cosmos/cosmos/raw/master/Cosmos_Token_Model.pdf).

### Bond your tokens

Bond your tokens to a validator candidate with the following command:

```
gaiacli delegate --amount=10steak --address-delegator=<your_address> --address-candidate=<bonded_validator_address> --name=<key_name> --chain-id=<name_of_testnet_chain> --sequence=2
```

When tokens are bonded, they are pooled with all the other bonded tokens in the network. Validators and delegators obtain shares that represent their stake in this pool. 

### Unbond

If for any reason the validator misbehaves or you just want to unbond a certain amount of the bonded tokens:

```
gaiacli unbond --address-delegator=<your_address> --address-candidate=<bonded_validator_address> --shares=MAX --name=<key_name> --chain-id=<name_of_testnet_chain> --sequence=3
```

You can unbond a specific amount of`shares`\(eg:`12.1`\) or all of them \(`MAX`\).

You should now see the unbonded tokens reflected in your balance and in your delegator bond:

```
gaiacli account <your_address>
gaiacli delegator-bond --address-delegator=<your_address> --address-candidate=<bonded_validator_address> --chain-id=<name_of_testnet_chain>
```
