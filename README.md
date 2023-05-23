# libcryptoconditions [![Build Status](https://travis-ci.org/libscott/libcryptoconditions.svg?branch=komodo)](https://travis-ci.org/libscott/libcryptoconditions)

Interledger Crypto-Conditions in C, targeting spec [draft-thomas-crypto-conditions-03](https://tools.ietf.org/html/draft-thomas-crypto-conditions-03).

Features shared object and easy to use JSON api, as well as a command line interface written in Python.

## Quickstart

```shell
git clone --recursive https://github.com/royari/libcryptoconditions
cd libcryptoconditions
./autogen.sh
./configure
make
./cryptoconditions.py --help
```

## Status

JSON interface may not be particularly safe.

## Testing

After building:

`make test`

## Embedding

For the binary interface, see [cryptoconditions.h](./include/cryptoconditions.h).

To embed in other languages, the easiest way may be to call the JSON RPC method via FFI. This is how it looks in Python:

```python
import json
from ctypes import *

so = cdll.LoadLibrary('.libs/libcryptoconditions.so')
so.jsonRPC.restype = c_char_p

def call_cryptoconditions_rpc(method, params):
    out = so.jsonRPC(json.dumps({
        'method': method,
        'params': params,
    }))
    return json.loads(out)
```

## JSON methods

### encodeCondition

Encode a JSON condition to a base64 binary string

```shell
./cryptoconditions.py encodeCondition '{
    "type": "ed25519-sha-256",
    "publicKey": "11qYAYKxCrfVS_7TyWQHOg7hcvPapiMlrwIaaPcHURo"
}'
```
output
```
{
	"uri":	"ni:///sha-256;eZI5q6j8T_fqv7xMROaei9_tmTMk4S7WR5Kr4onPHV8?fpt=ed25519-sha-256&cost=131072",
	"bin":	"A4278020799239ABA8FC4FF7EABFBC4C44E69E8BDFED993324E12ED64792ABE289CF1D5F8103020000"
}
```

### decodeCondition

Decode a binary condition

```shell
./cryptoconditions.py decodeCondition '{
    "bin": "A4278020799239ABA8FC4FF7EABFBC4C44E69E8BDFED993324E12ED64792ABE289CF1D5F8103020000"
}'
```
output
```
{
	"uri":	"ni:///sha-256;eZI5q6j8T_fqv7xMROaei9_tmTMk4S7WR5Kr4onPHV8?fpt=ed25519-sha-256&cost=131072",
	"bin":	"A4278020799239ABA8FC4FF7EABFBC4C44E69E8BDFED993324E12ED64792ABE289CF1D5F8103020000",
	"condition":	{
		"type":	"(anon)",
		"fingerprint":	"eZI5q6j8T_fqv7xMROaei9_tmTMk4S7WR5Kr4onPHV8",
		"cost":	131072,
		"subtypes":	0
	}
}
```

### encodeFulfillment

Encode a JSON condition to a binary fulfillment. The condition must be fulfilled, that is,
it needs to have signatures present.

```shell
./cryptoconditions.py encodeFulfillment '{
     "type": "ed25519-sha-256",
     "publicKey": "E0x0Ws4GhWhO_zBoUyaLbuqCz6hDdq11Ft1Dgbe9y9k",
     "signature": "jcuovSRpHwqiC781KzSM1Jd0Qtyfge0cMGttUdLOVdjJlSBFLTtgpinASOaJpd-VGjhSGWkp1hPWuMAAZq6pAg"
 }'
 
```
output
```
{
    "fulfillment":	"A4648020134C745ACE0685684EFF306853268B6EEA82CFA84376AD7516DD4381B7BDCBD981408DCBA8BD24691F0AA20BBF352B348CD4977442DC9F81ED1C306B6D51D2CE55D8C99520452D3B60A629C048E689A5DF951A3852196929D613D6B8C00066AEA902"
}

```

### decodeFulfillment

Decode a binary fulfillment

```shell
./cryptoconditions.py decodeFulfillment '{
    "fulfillment": "A4648020134C745ACE0685684EFF306853268B6EEA82CFA84376AD7516DD4381B7BDCBD981408DCBA8BD24691F0AA20BBF352B348CD4977442DC9F81ED1C306B6D51D2CE55D8C99520452D3B60A629C048E689A5DF951A3852196929D613D6B8C00066AEA902"
}'
```

output
```
{
    "uri":	"ni:///sha-256;j37bc776B2ydLAYpkBfGrhALhc9BBV1jesPDuJJyvzE?fpt=ed25519-sha-256&cost=131072",
    "bin":	"A42780208F7EDB73BEFA076C9D2C06299017C6AE100B85CF41055D637AC3C3B89272BF318103020000"
}
```

### verifyFulfillment

Verify a fulfillment against a message and a condition URL

```shell

./cryptoconditions.py verifyFulfillment '{
     "message": "48656c6c6f20576f726c642120436f6e646974696f6e7320617265206865726521",
     "fulfillment": "a4648020ec172b93ad5e563bf4932c70e1245034c35467ef2efd4d64ebf819683467e2bf8140b62291fad9432f8f298b9c4a4895dbe293f6ffda1a68dadf0ccdef5f47a0c7212a5fea3cda97a3f4c03ea9f2e8ac1cec86a51d452127abdba09d1b6f331c070a",
     "condition": "A427802053562115D5B494E23E4951773DB0CFE2DFE555E7E3FFEB41E4FD75CAF7C16A818103020000"
}'

```
output
```
{
    "valid": true
}
```

### signTreeEd25519 (output not verified)

Sign all ed25519 nodes in a condition tree

```shell
cryptoconditions signTreeEd25519 '{
    "condition": {
        "type": "ed25519-sha-256",
        "publicKey": "E0x0Ws4GhWhO_zBoUyaLbuqCz6hDdq11Ft1Dgbe9y9k",
    },
    "privateKey": "11qYAYKxCrfVS_7TyWQHOg7hcvPapiMlrwIaaPcHURo",
    "message": "",
}'
{
    "num_signed": 1,
    "condition": {
        "type": "ed25519-sha-256",
        "publicKey": "E0x0Ws4GhWhO_zBoUyaLbuqCz6hDdq11Ft1Dgbe9y9k",
        "signature": "jcuovSRpHwqiC781KzSM1Jd0Qtyfge0cMGttUdLOVdjJlSBFLTtgpinASOaJpd-VGjhSGWkp1hPWuMAAZq6pAg"
    }
}
```

### signTreeSecp256k1

Sign all secp256k1 nodes in a condition tree

```shell
cryptoconditions signTreeSecp256k1 '{
    "condition": {
        "type": "secp256k1-sha-256",
        "publicKey": "AmkauD4tVL5-I7NN9hE_A8SlA0WdCIeJe_1Nac_km1hr",
    },
    "privateKey": "Bxwd5hOLZcTvzrR5Cupm3IV7TWHHl8nNLeO4UhYfRs4",
    "message": "",
}'
{
    "num_signed": 1,
    "condition": {
        "type": "secp256k1-sha-256",
        "publicKey": "AmkauD4tVL5-I7NN9hE_A8SlA0WdCIeJe_1Nac_km1hr",
        "signature": "LSQLzZo4cmt04KoCdoFcbIJX5MZ9CM6324SqkdqV1PppfUwquiWa7HD97hf4jdkdqU3ep8ZS9AU7zEJoUAl_Gg"
    }
}
```
