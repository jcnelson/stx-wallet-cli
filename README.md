# wallet-cli

Stacks wallet swiss army-knife.

## Installation

Install the following [stacks.js](https://github.com/hirosystems/stacks.js/)
packages in your `$NODE_PATH`:

* `@stacks/common`
* `@stacks/transactions`
* `@stacks/network`

## Usage

```
$ ./wallet-cli help
Unrecognized subcommand 'undefined'
Usage: wallet-cli subcommand [options] args...
Where `subcommand` is any of the following:

  Accounts
     generate-account       Generate a wallet's secrets
     get-account            Get an account's balance from the node
     seed-phrase            Generate a wallet seed phrase

  Transactions
     token-transfer         Generate a token-transfer transaction
     contract-publish       Generate a contract-publish transaction
     contract-call          Generate a contract-call transaction

Run 'wallet-cli help subcommand' for subcommand-specific instructions.
Run 'wallet-cli help post-conditions' for how to craft post-conditions.
```

## Generating an account

```
$ ./wallet-cli help generate-account
Usage: wallet-cli generate-account [options] [index]
Where `index` refers to the nth account created from this seed phrase, starting from 0.
Values for 'options' include:
   -t, --testnet       Use testnet address format
   -s, --seed-phrase   12-word master secret. If '-' is passed, it will be read from stdin.

Outputs the account secrets as JSON
```

### Example

```
$ ./wallet-cli generate-account -s - 1 < ./test/test-seed-phrase.txt
{"stxPrivateKey":"2ca3ed6879438f7d3325159885c8f80f278563c03df042e3e799057f9dbc458a01","dataPrivateKey":"895c2534dfa5ebfc8932b770b31943bb39539a362a410a187d35f81d05046134","appsKey":"xprvA15Hx1JDPVphCBAocvNTX3eDDKPu7ghnRWAFg4HV7rkkcQ1yAS6pnTyz1cvFo8XzhUke92dP6W6BuiKybYNdhNZtQEAtpJLAy9pyJQbkaU6","salt":"9653dd9da8e8c9cebc70448052ec2119bfc71a095a3306583b15bb487d7b14a0","index":1,"stxAddress":"SPM8JZ8KXYR5YD99FKDN92S3SFZ2TKV5580CAJ55"}
```

## Getting an account

```
$ ./wallet-cli help get-account     
Usage: wallet-cli get-account [options] address
Where `address` is a standard or contract principal.
Values for 'options' include:
   -b, --at-block      Stacks chain tip block ID at which to run this function
   -N, --node          Stacks node URL. Default is http://seed-0.mainnet.stacks.co:20443

Outputs the account balance, locked tokens, unlock height, and nonce.
```

### Example

```
$ ./wallet-cli get-account -N "http://localhost:20443" SPM8JZ8KXYR5YD99FKDN92S3SFZ2TKV5580CAJ55
{"balance":"0","locked":"0","unlock_height":"0","nonce":"0"}
```

## Generating a seed phrase

```
$ ./wallet-cli help seed-phrase
Usage: wallet-cli seed-phrase

Outputs a 12-word BIP39 seed phrase
```

### Example

```
$ ./wallet-cli seed-phrase
"thought acquire slow fork intact devote six tattoo protect crazy panther lab bless city vacuum slush inside river scissors neither critic blue then model"
```

## Sending STX tokens

```
./wallet-cli help token-transfer
Usage: wallet-cli token-transfer [options] recipient amount
Where:
   `recipient` is a standard or contract principal
   `amount` is the number of uSTX

Values for 'options' include:
   -A, --account-index Index of the account to derive from the master key.
   -N, --node          URL to the node to use to look up the sending account nonce, if needed
   -a, --anchor-mode   "off-chain" or "on-chain" if required. Defaults to 'Any'.
   -f, --fee           uSTX transaction fee.
   -m, --memo          Set the memo field
   -n, --nonce         Use the given transaction nonce
   -s, --seed-phrase   12-word master secret. If '-' is passed, it will be read from stdin.
   -t, --testnet       Use testnet version bytes

Outputs the serialized transaction as a hex-encoded string.
```

### Example

```
$ ./wallet-cli token-transfer -N "http://localhost:20443" -m "abcde" -f 123 -n 456 -s - -A 1 -a "on-chain" \
ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS 789
< ./test/test-seed-phrase.txt
"0000000001040028897d13efb05f35297cdb548b23cbfe2d4f652a00000000000001c8000000000000007b00014ab2550377c630d37f96365357bce1317f8fb62823feb9bbab20ae52b4c752af4397565c79401cac4ed79dcbd4592cdd1dea30f494578b75d2179dc84be24f5801020000000000051a634d03a67375b7d53fbe11d63835a5df6d989bc5000000000000031561626364650000000000000000000000000000000000000000000000000000000000"
```

## Publishing a contract

```
$ ./wallet-cli help contract-publish
Usage: wallet-cli contract-publish [options] contract-name contract-src
Where:
   `contract-name` is the name of the contract
   `contract-src` is the path to the contract code on disk

Values for 'options' include:
   -A, --account-index Index of the account to derive from the master key.
   -N, --node          URL to the node to use to look up the sending account nonce, if needed
   -a, --anchor-mode   "off-chain" or "on-chain" if required. Defaults to 'Any'.
   -f, --fee           uSTX transaction fee.
   -n, --nonce         Use the given transaction nonce
   -p, --post-condition
   -s, --seed-phrase   12-word master secret. If '-' is passed, it will be read from stdin.
   -t, --testnet       Use testnet version bytes

Outputs the serialized transaction as a hex-encoded string.
```

### Example

```
$ ./wallet-cli contract-publish -N "http://52.0.54.100:20443" -f 123 -n 456 -s - -A 1 -a "on-chain" \
-p "allow \
stx ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS <= 1 \
ft ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS >= ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS.bar stackaroo 2 \
nft ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS keeps ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS.baz quux 09" \
foo /tmp/hello.clar \
< ./test/test-seed-phrase.txt
"0000000001040028897d13efb05f35297cdb548b23cbfe2d4f652a0000000000000000000000000000007b00004ca81cbc4221c03ed656550057bb7d1733485dd0b634836d9cf6d2584acd5dd031415c01375ea0383e05fa150faff92478ad55bdf1fbaee7574025721d41cb4c01010000000300021a634d03a67375b7d53fbe11d63835a5df6d989bc505000000000000000101021a634d03a67375b7d53fbe11d63835a5df6d989bc51a634d03a67375b7d53fbe11d63835a5df6d989bc50362617209737461636b61726f6f02000000000000000202021a634d03a67375b7d53fbe11d63835a5df6d989bc51a634d03a67375b7d53fbe11d63835a5df6d989bc50362617a047175757809110103666f6f00000016287072696e74202268656c6c6f20776f726c6422290a"
```

## Calling a public contract function

```
$ ./wallet-cli help contract-call
Usage: wallet-cli contract-call [options] contract-id function-name [arg...]
Where:
   `contract-id` is the fully-qualified address of the contract
   `function-name` is the name of the function to call
   `[arg ...]` is a list of zero or more arguments, as hex-encoded serialized Clarity values

Values for 'options' include:
   -A, --account-index Index of the account to derive from the master key.
   -N, --node          URL to the node to use to look up the sending account nonce, if needed
   -a, --anchor-mode   "off-chain" or "on-chain" if required. Defaults to 'Any'.
   -f, --fee           uSTX transaction fee.
   -n, --nonce         Use the given transaction nonce
   -p, --post-conds    Post-conditions. Run `help post-conditions` for details.
   -s, --seed-phrase   12-word master secret. If '-' is passed, it will be read from stdin.
   -t, --testnet       Use testnet version bytes

Outputs the serialized transaction as a hex-encoded string.
```

**Tip:** You can use the
[stacks-node-cli](https://github.com/jcnelson/stacks-node-cli) to encode Clarity
values from a human-readable shell-friendly form to hex.

### Example

```
$ ./wallet-cli contract-call -N "http://52.0.54.100:20443" -f 123 -n 456 -s - -A 1 -a "on-chain" \
ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS.foo bar-func 0100000000000000000000000000000003 09 \
< ./test/test-seed-phrase.txt
"0000000001040028897d13efb05f35297cdb548b23cbfe2d4f652a0000000000000000000000000000007b000069a1f0411b66397399ed30075753f4012284889afbaab5f59c95aec343ac434d715a892cd7643b4398786eb1cbcf7912f0a81b89a051fc68e32c3bd2a2a42888010200000000021a634d03a67375b7d53fbe11d63835a5df6d989bc503666f6f086261722d66756e6300000002010000000000000000000000000000000309"
```

## Post-conditions

The `contract-publish` and `contract-call` commands take a list of
post-conditions:

```
$ ./wallet-cli help post-conditions
Post-conditions help

Post-conditions are encoded as a sequence of words as follows:

     post-condition = mode conditions
               mode = "allow" | "deny"
         conditions = type parameters
               type = "stx" | "ft" | "nft"
         parameters = stxParameters | ftParameters | nftParameters
      stxParameters = principal fungibleCode amount
       ftParameters = principal fungibleCode contractID name amount
      nftParameters = principal nftCode contractID name value
          principal = standardPrincipal | contractID
  standardPrincipal = (a standard STX principal)
       fungibleCode = "<" | "<=" | "==" | ">=" | ">"
            nftCode = "sends" | "keeps"
         contractID = (a fully-qualified STX contract principal)
               name = (a valid FT or NFT name)
             amount = (an unsigned integer)
              value = (a hex-encoded serialized Clarity value)

Examples:

  "Allow-all; `ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS` must spend less than or equal to 1 uSTX"

      `-p "allow stx ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS <= 1`

  "Allow-all; `ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS` must spend greater than or equal to 2 `stackaroos` from the
  from the `ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS.bar` contract

      `-p "allow ft ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS >= ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS.bar stackaroo 2"`

  "Allow-all; `ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS` does not send the NFT value `none` from the
  NFT `quux` defined in contract `ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS.baz`

      `-p "allow nft ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS keeps ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS.baz quux 09"`

Note that post-conditions are concatenative.  For example, you can combine all of the above post-conditions as follows:

      `-p "allow \
           stx ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS <= 1 \
           ft ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS >= ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS.bar stackaroo 2 \
           nft ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS keeps ST1HMT0X6EDTVFN9ZQR8XCE1NMQFPV64VRQWXV8WS.baz quux 09"
```
