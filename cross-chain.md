Cross-Chain NFTs using CrossTalk:

Cross-chain nfts: Transfering of NFT from one blockchain to another blockchain .

CrossTalk: RouterProtocol's in-house library to perform this task(ie. Cross-chaining / briding NFTs).

We will be using ERC1155 token to make truly cross-chain NFTs.

𝐈𝐦𝐩𝐨𝐫𝐭 𝐑𝐞𝐪. 𝐜𝐨𝐧𝐭𝐫𝐚𝐜𝐭𝐬:

pragma solidity ^0.8.0;
​
import "@routerprotocol/router-crosstalk/contracts/RouterCrossTalk.sol";
import "@openzeppelin/contracts/token/ERC1155/ERC1155.sol";
import "@openzeppelin/contracts/token/ERC20/IERC20.sol";
​
We are defining the solidity version.

After that, we are importing RouterCrossTalk contract along with OPenzeppelin's ERC1155 contract and Interface of ERC20 token.

𝐈𝐧𝐢𝐭𝐢𝐚𝐥𝐢𝐳𝐞 𝐚 𝐍𝐞𝐰 𝐂𝐨𝐧𝐭𝐫𝐚𝐜𝐭:

contract CrossChainERC1155 is ERC1155, RouterCrossTalk { 
    address public owner;
    uint256 private _crossChainGasLimit;
    
    constructor(string memory uri_, address genericHandler_)
    ERC1155(uri_) RouterCrossTalk(genericHandler_) {
        owner = msg.sender;
    }


We are creating a new Contract named CrossChainERC1155 and inheriting ERC1155, RouterCrossTalk 
and declaring 2 variables 
one as owner of the contract, others as 
gasLimit while transfering NFT from one chain to other.
and passing the constructor arguments “uri_” and “genericHandler_”.

𝐒𝐞𝐭𝐭𝐢𝐧𝐠 𝐀𝐝𝐦𝐢𝐧𝐢𝐬𝐭𝐫𝐚𝐭𝐢𝐯𝐞 𝐟𝐮𝐧𝐜𝐭𝐢𝐨𝐧𝐬:

 
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

In the above functions we are setting admin powers to the functions by which only the owner will be able to modify that functions because we dont want to other developers ot modify them.

functions like, 
setFeesToken -> in this function we will be deciding in which token we will be charging the fees and we only want admin to set this.

_𝐬𝐞𝐧𝐝𝐂𝐫𝐨𝐬𝐬𝐂𝐡𝐚𝐢𝐧 𝐅𝐮𝐧𝐜𝐭𝐢𝐨𝐧:

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

𝐒𝐞𝐧𝐝𝐂𝐫𝐨𝐬𝐬𝐂𝐡𝐚𝐢𝐧 𝐟𝐮𝐧𝐜𝐭𝐢𝐨𝐧:

It is the heart of our contract because in this function we are deciding destination, chainId, receiver address, id number, amount to be transfered, crossChainGasPrice(amount of gas to be spent when transfering from sender to receiver).

It returns bool and byte datatype.

It calls _burnBatch() by which we are deleting the particular id and the amount that has been transferred( or oppn to happen), in burn batch we will be burning more than 2 ids because of which it reduces gas cost and makes process easier.

After that we are hashing the receiveCrossChain function and type casting them into bytes4 type as keccak256 hashing returns 32 bytes data & we required bytes4 type of data.

In this line we are encoding recipient,ids,amounts and data, storing them in data of bytes datatype.

In this line we are collecting success(boolean) and hash(bytes) from routerSend() which returns bunch of data but we need just 2 variables so in Soldity using the above method we can perform this task.

And at end returning success, and hash.

_𝐫𝐨𝐮𝐭𝐞𝐫𝐒𝐲𝐧𝐜𝐇𝐚𝐧𝐝𝐥𝐞𝐫 𝐅𝐮𝐧𝐜𝐭𝐢𝐨𝐧:

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


In this function we are syncing with the data that is about to pass. 

WTH do I mean by above line is ,

In this function we are overriding the RouterCrossTalk's contract function's logic by placing it with our requirements,

𝐫𝐞𝐜𝐞𝐢𝐯𝐞𝐂𝐫𝐨𝐬𝐬𝐂𝐡𝐚𝐢𝐧 𝐅𝐮𝐧𝐜𝐭𝐢𝐨𝐧:

  function receiveCrossChain(
        address _recipient,
        uint256[] memory _ids,
        uint256[] memory _amounts,
        bytes memory _data
    ) external isSelf returns (bool) {
        _mintBatch(_recipient, _ids, _amounts, _data);
        return true;
    }

This function will burn the tokens in source chain after sending the respective tokens to the destination chain.
It accepts recipients adress, array of id's, array of amounts and data and returns true if the mint is successful or else false.

𝐫𝐞𝐩𝐥𝐚𝐲𝐓𝐫𝐚𝐧𝐬𝐟𝐞𝐫𝐂𝐫𝐨𝐬𝐬𝐂𝐡𝐚𝐢𝐧 𝐅𝐮𝐧𝐜𝐭𝐢𝐨𝐧:

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

This function is used to revert back the transaction if the source chain does'nt provide sufficient gas while executing the transaction.

𝐑𝐞𝐟𝐞𝐫 𝐝𝐨𝐜𝐬: 

https://dev.routerprotocol.com
https://dev.routerprotocol.com/crosstalk-library/cross-chain-nfts-using-crosstalk/replaytransfercrosschain-function
