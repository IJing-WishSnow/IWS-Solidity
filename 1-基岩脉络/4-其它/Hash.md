# Hash

## Hash的性质

一个好的哈希函数应该具有以下几个特性：

单向性：从输入的消息到它的哈希的正向运算简单且唯一确定，而反过来非常难，只能靠暴力枚举。
灵敏性：输入的消息改变一点对它的哈希改变很大。
高效性：从输入的消息到哈希的运算高效。
均一性：每个哈希值被取到的概率应该基本相等。
抗碰撞性：
弱抗碰撞性：给定一个消息x，找到另一个消息x'，使得hash(x) = hash(x')是困难的。
强抗碰撞性：找到任意x和x'，使得hash(x) = hash(x')是困难的。

## Hash的应用

- 生成数据唯一标识
- 加密签名
- 安全加密

## Keccak256

哈希 = keccak256(数据);

    ```solidity
    function hash(
        uint _num,
        string memory _string,
        address _addr
        ) public pure returns (bytes32) {
        return keccak256(abi.encodePacked(_num, _string, _addr));
    }
    ```

sha3由keccak标准化而来，在很多场合下Keccak和SHA3是同义词，但在2015年8月SHA3最终完成标准化时，NIST调整了填充算法。所以SHA3就和keccak计算的结果不一样，这点在实际开发中要注意。  
以太坊在开发的时候sha3还在标准化中，所以采用了keccak，所以Ethereum和Solidity智能合约代码中的SHA3是指Keccak256，而不是标准的NIST-SHA3，为了避免混淆，直接在合约代码中写成Keccak256是最清晰的。
