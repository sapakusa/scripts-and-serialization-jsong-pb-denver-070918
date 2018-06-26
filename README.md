# Script [not working]

The ability to lock and unlock coins is at the heart of what it means to transfer Bitcoin.

### Mechanics of SCRIPT

If you are confused about what a "smart contract" is, don't worry. "Smart contract" is a fancy way of saying "programmable" and the "smart contract language" is simply a programming language. SCRIPT is the smart contract langauge, or the programming language used to express how bitcoins are to be transferred.

Think of a personal check. In a sense, a personal check is a type of contract. A personal check is an agreement to transfer some amount of money from one person to another. Bitcoin has the digital equivalent of a contract in SCRIPT.

SCRIPT is a limited programming language in the sense that it doesn't have certain features. Specifically, it does not have any mechanism for loops and is not Turing complete.

What you are required to do as part of a transaction is to assign Bitcoins to a *locking* script. The locking script is what's specified in ScriptPubKey (see chapter 5). Think of this as the locked box where some money is deposited which only the person with the key to the box can open. The money inside, of course, can only be accessed by someone with the key.

The actual unlocking of bitcoin is done in the ScriptSig field and proves ownership of the locked box in order to spend the funds.

#### Exercise: Determine a ScriptSig that will satisfy this scriptPubKey:
```
767695935687
```
#### Hint: use the Script.parse method

```python
# Exercise 1.1

from script import Script

hex_script = '767695935687'

# bytes.fromhex the script
# parse the script
```

### Exercise 2

#### 2.1. Determine what this scriptPubKey is doing:
```
6e879169a77ca787
```

#### Hint: Use the Script.parse method and look up what various OP codes do here:
#### https://en.bitcoin.it/wiki/Script


```python
# Exercise 2.1

from script import Script

hex_script = '6e879169a77ca787'

# bytes.fromhex the script
# parse the script
```

### Test Driven Exercise

```python
import requests
from io import BytesIO
from tx import Tx, TxIn, TxOut
from helper import (
    encode_varint,
    int_to_little_endian,
    little_endian_to_int
)

class TxIn(TxIn):

    def serialize(self):
        '''Returns the byte serialization of the transaction input'''
        # serialize prev_tx, little endian
        result = self.prev_tx[::-1]
        # serialize prev_index, 4 bytes, little endian
        result += int_to_little_endian(self.prev_index, 4)
        # get the scriptSig ready (use self.script_sig.serialize())
        raw_script_sig = self.script_sig.serialize()
        # encode_varint on the length of the scriptSig
        result += encode_varint(len(raw_script_sig))
        # add the scriptSig
        result += raw_script_sig
        # serialize sequence, 4 bytes, little endian
        result += int_to_little_endian(self.sequence, 4)
        return result

class Tx(Tx):

    def serialize(self):
        '''Returns the byte serialization of the transaction'''
        # serialize version (4 bytes, little endian)
        result = int_to_little_endian(self.version, 4)
        # encode_varint on the number of inputs
        result += encode_varint(len(self.tx_ins))
        # iterate inputs
        for tx_in in self.tx_ins:
            # serialize each input
            result += tx_in.serialize()
        # encode_varint on the number of inputs
        result += encode_varint(len(self.tx_outs))
        # iterate outputs
        for tx_out in self.tx_outs:
            # serialize each output
            result += tx_out.serialize()
        # serialize locktime (4 bytes, little endian)
        result += int_to_little_endian(self.locktime, 4)
        return result

class TxOut(TxOut):

    def serialize(self):
        '''Returns the byte serialization of the transaction output'''
        # serialize amount, 8 bytes, little endian
        result = int_to_little_endian(self.amount, 8)
        # get the scriptPubkey ready (use self.script_pubkey.serialize())
        raw_script_pubkey = self.script_pubkey.serialize()
        # encode_varint on the length of the scriptPubkey
        result += encode_varint(len(raw_script_pubkey))
        # add the scriptPubKey
        result += raw_script_pubkey
        return result
```
