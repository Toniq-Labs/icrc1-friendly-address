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
principal: # this would still format correctly
```

# Known protocols
Some known protocols can be registered to allow for more friendly addresses. For example, the protocol `volt` is regsitered to the canisterId `[todo]`. Because Volt is a cansiter based account protocol, each user is assigned a subaccount linked to their principal (in fact it's their principal represented in hex format). So these addresses could look like:
```
volt:283ecaff5fd8481ed2ee81e418b69168165697521b42257ab3ab468da05a6f94
```

