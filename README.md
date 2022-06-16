# Cardano-SimpleToken

This project represents an implementation of a native asset, in this case, a fungible token, on the Cardano blockchain. Native assets in Cardano are created via Command Line Interface. For this example we generate 1000 Simple Tokens (STK) and transfer 300 tokens from Alice to Bob. Steps needed are taken from the Cardano official page and are listed below. 

**NOTE!**  
These is not the full tutorial on how to mint native asset, but rather a guide for reader to be able to coprehend how the files in the repository are created. Many steps were omitted as it would be copying the official guide and parameters adaptation. For full tutorial and more detailed steps see [Cardano Official Documentation](https://developers.cardano.org/docs/native-tokens/minting/)

## Setup 

### Running Cardano node

```bash
$HOME/.local/bin/cardano-node run \
 --topology config/testnet-topology.json \
 --database-path db \
 --socket-path $HOME/TESTNET_NODE/socket/node.socket \
 --port 3001 \
 --config config/testnet-config.json
```
Setting the variable to the path of socket parameter

```bash
export CARDANO_NODE_SOCKET_PATH="$HOME/TESTNET_NODE/socket/node.socket"
```
### Defining variables to improve readability 

```bash
testnet="--testnet-magic 1097911063"
tokenname=$(echo -n "SimpleToken" | xxd -ps | tr -d '\n')
tokenamount="1000"
output="0" 
```
### Generating key and addresses

Creating keys and address for Alice

```bash
cardano-cli address key-gen --verification-key-file payment.vkey --signing-key-file payment.skey

cardano-cli address build --payment-verification-key-file payment.vkey --out-file payment.addr $testnet

alice=$(cat payment.addr)

```

Creating keys and address for Bob

```bash
cardano-cli address key-gen --verification-key-file payment2.vkey --signing-key-file payment2.skey

cardano-cli address build --payment-verification-key-file payment2.vkey --out-file payment2.addr $testnet

bob=$(cat payment2.addr)

```

### Funding Alice's address

To fund Alice's address we used [testnet faucet](https://developers.cardano.org/docs/integrate-cardano/testnet-faucet/).

### Generating the policy

```bash
cardano-cli address key-gen \
    --verification-key-file policy/policy.vkey \
    --signing-key-file policy/policy.skey

touch policy/policy.script && echo "" > policy/policy.script

echo "{" >> policy/policy.script 
echo "  \"keyHash\": \"$(cardano-cli address key-hash --payment-verification-key-file policy/policy.vkey)\"," >> policy/policy.script 
echo "  \"type\": \"sig\"" >> policy/policy.script 
echo "}" >> policy/policy.script
```
### Asset minting

```bash
cardano-cli transaction policyid --script-file ./policy/policy.script > policy/policyID

cardano-cli transaction build-raw \
 --fee $fee \
 --tx-in $txhash#$txix \
 --tx-out $alice+$output+"$tokenamount $policyid.$tokenname \
 --mint="$tokenamount $policyid.$tokenname + $tokenamount" \
 --minting-script-file policy/policy.script \
 --out-file matx.raw 

cardano-cli transaction sign  \
--signing-key-file payment.skey  \
--signing-key-file policy/policy.skey  \
$testnet --tx-body-file matx.raw  \
--out-file matx.signed

cardano-cli transaction submit --tx-file matx.signed $testnet
```

### Sending tokens to Bob

#### Defining variables
```bash
fee="0"
receiver=$bob
receiver_output="300"
txhash=""
txix=""
funds="Amout of lovelace" 
```

The transaction creation steps remain the same as described above
