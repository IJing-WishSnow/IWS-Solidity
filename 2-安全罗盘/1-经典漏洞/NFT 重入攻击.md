# NFT 重入攻击


NFT标准（ERC721/ERC1155）为了防止用户误把资产转入黑洞而加入了安全转账：如果转入地址为合约，则会调用该地址相应的检查函数，确保它已准备好接收NFT资产。例如 ERC721 的 safeTransferFrom() 函数会调用目标地址的 onERC721Received() 函数，而黑客可以把恶意代码嵌入其中进行攻击。

ERC721 和 ERC1155 有潜在重入风险的函数：
ERC721:
safeTransferFrom
_safeTransfer
_safeMint
_checkOnERC721Received

ERC1155:
safeTransferFrom
_safeTransferFrom
safeBatchTransferFrom
_safeBatchTransferFrom
_mint
_mintBatch
_doSafeTransferAcceptanceCheck
_doSafeBatchTransferAcceptanceCheck

主要有两种办法来预防重入攻击漏洞： 检查-影响-交互模式（checks-effect-interaction）和重入锁。

检查-影响-交互模式：它强调编写函数时，要先检查状态变量是否符合要求，紧接着更新状态变量（例如余额），最后再和别的合约交互。我们可以用这个模式修复有漏洞的mint()函数:
    function mint() payable external {
        // 检查是否mint过
        require(mintedAddress[msg.sender] == false);
        // 增加total supply
        totalSupply++;
        // 记录mint过的地址
        mintedAddress[msg.sender] = true;
        // mint
        _safeMint(msg.sender, totalSupply);
    }
重入锁：它是一种防止重入函数的修饰器（modifier）。建议直接使用OpenZeppelin提供的ReentrancyGuard
