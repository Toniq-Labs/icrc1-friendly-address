# icrc1-friendly-address
ICRC1 does not understand `AccountIdentifier` which is used by the ICP ledger and other token standards. The benefit of the `AccountIdentifier` is the ability to control multiple addresses for a single principal (via subaccounts). The downside is that `AccountIdentifier` it generated using a hash function, so is one way.

ICRC1 uses an "unhashed" representation of this same concept
```
type Subaccount = [Nat8];
type Account = {
  owner : Principal;
  subaccount : ?Subaccount;
};
```

Most current implementations of ICRC1 treat the standard as a "principal" only standard, ignoring the subaccount. This is understandable as additional work is required (now when sending/sharing your ICRC1 address, you would need to share both the principal and the subaccount). The need for a friendly format for this address is apparent as per this [forum thread](https://forum.dfinity.org/t/icrc-1-account-human-readable-format/14682/38)

# Our Proposal
We propose a simple address format, and JS tools for encoding and decoding.
```
principal[:subaccount as hex]
```
The principal is any valid principal (including canisters), which allows for backward compatability. 

The subaccount is optional, and represented in hex format with leading 0's trimmed (to save space). The max size of the subaccount is 32 bytes, or 64 hex characters. By default, if not subaccount is present, we use the null/0 subaccount.

```
# These are all the same
principal
principal:00 # is the same as principal (leading 0s are trimmed)
principal:0 # this would be valid
principal: # this would still format correctly (default null/0 subaccount)
```

# Known protocols
Some known protocols can be registered to allow for more friendly addresses. For example, the protocol `volt` is regsitered to the canisterId `aclt4-uaaaa-aaaak-qb4zq-cai`. Because Volt is a cansiter based account protocol, each user is assigned a subaccount linked to their principal (in fact it's their principal represented in hex format). So these addresses could look like:
```
# these are the same
volt:ff5fd8481ed2ee81e418b69168165697521b42257ab3ab468da05a6f94
aclt4-uaaaa-aaaak-qb4zq-cai:ff5fd8481ed2ee81e418b69168165697521b42257ab3ab468da05a6f94
aclt4-uaaaa-aaaak-qb4zq-cai:000000ff5fd8481ed2ee81e418b69168165697521b42257ab3ab468da05a6f94
```

#Implementation
Here is a basic JS implementation to convert an address in this format to a valid JS Candid `Account` object.
```
const Principal = require("@dfinity/principal").Principal;
const PROTOCOLS = {
  "volt" : "aclt4-uaaaa-aaaak-qb4zq-cai",
};
const toHexString = (byteArray) => {
    return Array.from(byteArray, (byte) => ('0' + (byte & 0xff).toString(16)).slice(-2)).join('');
};
const fromHexString = (hex) => {
    if (hex.substr(0, 2) === '0x') hex = hex.substr(2);
    for (var bytes = [], c = 0; c < hex.length; c += 2) bytes.push(parseInt(hex.substr(c, 2), 16));
    return bytes;
};
const getSubaccountFromHex = (hex) => {
  const dec = fromHexString(hex);
  return Array(32-dec.length).fill(0).concat(dec);
};
const IcrcAccountFromAddress = address => {
  const decoded = address.split(":");
  const principalText = (protocols.hasOwnProperty(decoded[0]) ? PROTOCOLS[decoded[0]] : decoded[0]);
  const subaccount = (decoded.length > 1 ? [getSubaccountFromHex(decoded[1])]: []);
  return {owner : Principal.fromText(principalText), subaccount : subaccount};
};

// Examples

console.log(IcrcAccountFromAddress("volt"));
console.log(IcrcAccountFromAddress("volt:0"));
console.log(IcrcAccountFromAddress("volt:"));
console.log(IcrcAccountFromAddress("volt:e5c6a05f916d8408d4bc3f8a6e920cf9330ad3344d5505c534e6048e02"));
```

