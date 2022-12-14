Cross-Chain NFTs using CrossTalk:

Cross-chain NFTs: Transferring of NFT from one blockchain to another blockchain.

CrossTalk: RouterProtocol's in-house library to perform this task(ie. Cross-chaining / briding NFTs).

We will be using the ERC1155 token to make truly cross-chain NFTs.

๐๐ฆ๐ฉ๐จ๐ซ๐ญ ๐๐๐ช. ๐๐จ๐ง๐ญ๐ซ๐๐๐ญ๐ฌ:

pragma solidity ^0.8.0;
โ

import "@routerprotocol/router-crosstalk/contracts/RouterCrossTalk.sol";

import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";

import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
โ
We are defining the solidity version.

After that, we are importing the RouterCrossTalk contract along with Openzeppelin's ERC1155 contract and the Interface of the ERC20 token.

๐๐ง๐ข๐ญ๐ข๐๐ฅ๐ข๐ณ๐ ๐ ๐๐๐ฐ ๐๐จ๐ง๐ญ๐ซ๐๐๐ญ:

contract CrossChainERC1155 is ERC1155, RouterCrossTalk { 
    address public owner;
    uint256 private _crossChainGasLimit;
    
    constructor(string memory uri_, address genericHandler_)
    ERC1155(uri_) RouterCrossTalk(genericHandler_) {
        owner = msg.sender;
    }


We are creating a new Contract named CrossChainERC1155 and inheriting ERC1155, RouterCrossTalk 
and declaring 2 variables 
one as the owner of the contract, others as 
gasLimit while transferring NFT from one chain to other.
and passing the constructor arguments โuri_โ and โgenericHandler_โ.

๐๐๐ญ๐ญ๐ข๐ง๐  ๐๐๐ฆ๐ข๐ง๐ข๐ฌ๐ญ๐ซ๐๐ญ๐ข๐ฏ๐ ๐๐ฎ๐ง๐๐ญ๐ข๐จ๐ง๐ฌ:

 
   /**
     * @notice setLinker Used to set address of linker, this can only be set by Admin
     * @param _linker Address of the linker
     */
    function setLinker(address _linker) external onlyOwner {
        setLink(_linker);
    }

    /**
     * @notice _approveFees To approve handler to deduct fees from source contract, this can only be set by Admin
     * @param _feeToken Address of the feeToken
     * @param _amount Amount to be approved
     */
    function _approveFees(address _feeToken, uint256 _amount)
        external
        onlyOwner
    {
        approveFees(_feeToken, _amount);
    }

    /**
     * @notice setFeesToken To set the fee token in which fee is desired to be charged, this can only be set by Admin
     * @param _feeToken Address of the feeToken
     */
    function setFeesToken(address _feeToken) external onlyOwner {
        setFeeToken(_feeToken);
    }
    
    /**
     * @notice setCrossChainGasLimit Used to set CrossChainGasLimit, this can only be set by Admin
     * @param _gasLimit Amount of gasLimit that is to be set
     */
    function setCrossChainGasLimit(uint256 _gasLimit) external onlyOwner {
        _crossChainGasLimit = _gasLimit;
    }

    /**
     * @notice fetchCrossChainGasLimit Used to fetch CrossChainGasLimit
     * @return crossChainGasLimit that is set
     */
    function fetchCrossChainGasLimit() external view returns (uint256) {
        return _crossChainGasLimit;
    }

The above functions owner will only be able to modify those functions because we don't want other developers to modify them.

functions like, 
setFeesToken -> in this function we will be deciding in which token we will be charging the fees and we only want the admin to set this.

_๐ฌ๐๐ง๐๐๐ซ๐จ๐ฌ๐ฌ๐๐ก๐๐ข๐ง ๐๐ฎ๐ง๐๐ญ๐ข๐จ๐ง:

function _sendCrossChain(
        uint8 _destChainID,
        address _recipient,
        uint256[] memory _ids,
        uint256[] memory _amounts,
        bytes memory _data,
        uint256 _crossChainGasPrice
    ) internal returns (bool, bytes32) {
        _burnBatch(msg.sender, _ids, _amounts);
        bytes4 _selector = bytes4(
            keccak256("receiveCrossChain(address,uint256[],uint256[],bytes)")
        );
        bytes memory data = abi.encode(_recipient, _ids, _amounts, _data);
        (bool success, bytes32 hash) = routerSend(
            _destChainID,
            _selector,
            data,
            _crossChainGasLimit,
            _crossChainGasPrice
        );

        return (success, hash);
    }

๐๐๐ง๐๐๐ซ๐จ๐ฌ๐ฌ๐๐ก๐๐ข๐ง ๐๐ฎ๐ง๐๐ญ๐ข๐จ๐ง:

It is the heart of our contract because in this function we are deciding destination, chainId, receiver address, id number, amount to be transferred, and crossChainGasPrice(amount of gas to be spent when transferring from sender to receiver).

It returns the bool and byte datatype.

It calls _burnBatch() by which we are deleting the particular id and the amount that has been transferred, in the burn batch, we will be burning multiple ids which reduce gas cost and make the process easier.

After that, we are hashing the receiveCrossChain function and type casting them into bytes4 type as keccak256 hashing returns 32 bytes data & we required bytes4 type of data.

In this line we are encoding recipient, ids, amounts, and data, storing them in data of bytes datatype.

In this line, we are collecting success(boolean) and hash(bytes) from routerSend() which returns a bunch of data but we need just 2 variables so in Solidity using the above method we can perform this task.

And at the end returning success and hash.

_๐ซ๐จ๐ฎ๐ญ๐๐ซ๐๐ฒ๐ง๐๐๐๐ง๐๐ฅ๐๐ซ ๐๐ฎ๐ง๐๐ญ๐ข๐จ๐ง:

function _routerSyncHandler(bytes4 _selector, bytes memory _data)
        internal
        virtual
        override
        returns (bool, bytes memory)
    {
        (address _recipient, uint256[] memory _ids, uint256[] memory _amounts, bytes memory data) = abi.decode(
            _data,
            (address, uint256[], uint256[], bytes)
        );
        (bool success, bytes memory returnData) = address(this).call(
            abi.encodeWithSelector(_selector, _recipient, _ids, _amounts, data)
        );
        return (success, returnData);
    }


In this function, we are syncing with the data that is about to pass. 

WTH do I mean by the above line is,

In this function, we are overriding the RouterCrossTalk's contract function's logic by placing it with our requirements,

๐ซ๐๐๐๐ข๐ฏ๐๐๐ซ๐จ๐ฌ๐ฌ๐๐ก๐๐ข๐ง ๐๐ฎ๐ง๐๐ญ๐ข๐จ๐ง:

  function receiveCrossChain(
        address _recipient,
        uint256[] memory _ids,
        uint256[] memory _amounts,
        bytes memory _data
    ) external isSelf returns (bool) {
        _mintBatch(_recipient, _ids, _amounts, _data);
        return true;
    }

This function will burn the tokens in the source chain after sending the respective tokens to the destination chain.
It accepts the recipient's address, an array of IDs, an array of amounts, and data and returns true if the mint is successful or else false.

๐ซ๐๐ฉ๐ฅ๐๐ฒ๐๐ซ๐๐ง๐ฌ๐๐๐ซ๐๐ซ๐จ๐ฌ๐ฌ๐๐ก๐๐ข๐ง ๐๐ฎ๐ง๐๐ญ๐ข๐จ๐ง:

    /**
     * @notice replayTransaction Used to replay the transaction if it failed due to low gaslimit or gasprice
     * @param hash Hash returned by transferCrossChain function
     * @param crossChainGasLimit new crossChainGasLimit
     * @param crossChainGasPrice new crossChainGasPrice
     * NOTE gasLimit and gasPrice passed in this function should be greater than what was passed earlier
     */
    function replayTransaction(
        bytes32 hash,
        uint256 crossChainGasLimit,
        uint256 crossChainGasPrice
    ) internal {
        routerReplay(hash, crossChainGasLimit, crossChainGasPrice);
    }

This function is used to revert the transaction if the source chain doesn't provide sufficient gas while executing the transaction.

๐๐๐๐๐ซ ๐๐จ๐๐ฌ: 

https://dev.routerprotocol.com

https://dev.routerprotocol.com/crosstalk-library/cross-chain-nfts-using-crosstalk/replaytransfercrosschain-function
