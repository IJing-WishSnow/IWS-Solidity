# ABI编码

## abi.encode

```solidity
function encode() public view returns(bytes memory result) {
    result = abi.encode(x, addr, name, array);
}

// 000000000000000000000000000000000000000000000000000000000000000a    // x
// 0000000000000000000000007a58c0be72be218b41c608b7fe7c5bb630736c71    // addr
// 00000000000000000000000000000000000000000000000000000000000000a0    // name 参数的偏移量
// 0000000000000000000000000000000000000000000000000000000000000005    // array[0]
// 0000000000000000000000000000000000000000000000000000000000000006    // array[1]
// 0000000000000000000000000000000000000000000000000000000000000004    // name 参数的长度为4字节
// 3078414100000000000000000000000000000000000000000000000000000000    // name

// 偏移量指向的是变量值的存储位置。
// 在这个编码中：
// 00000000000000000000000000000000000000000000000000000000000000a0 是 name 的偏移量

// 这个偏移量指向后面 name 实际值的存储位置（从开头算起第 160 字节处）
// 在偏移量指向的位置存储的是 name 变量的值，不是变量名
// 偏移量概念：动态类型数据的实际值存储在编码结果的后面部分，前面用偏移量来指向实际值的位置。
```

## abi.encodePacked

```solidity

// 将给定参数根据其所需最低空间编码。它类似 abi.encode，但是会把其中填充的很多0省略。比如，只用1字节来编码uint8类型。
// 当想省空间，并且不与合约交互的时候，可以使用abi.encodePacked，例如算一些数据的hash时。
// 需要注意，abi.encodePacked因为不会做填充，所以不同的输入在拼接后可能会产生相同的编码结果，导致冲突，这也带来了潜在的安全风险。

function encodePacked() public view returns(bytes memory result) {
    result = abi.encodePacked(x, addr, name, array);
}

// 编码的结果为0x000000000000000000000000000000000000000000000000000000000000000a7a58c0be72be218b41c608b7fe7c5bb630736c713078414100000000000000000000000000000000000000000000000000000000000000050000000000000000000000000000000000000000000000000000000000000006，
// 由于abi.encodePacked对编码进行了压缩，长度比abi.encode短很多。
```

## abi.encodeWithSignature

```solidity

// 与abi.encode功能类似，只不过第一个参数为函数签名，比如"foo(uint256,address,string,uint256[2])"。
// 当调用其他合约的时候可以使用。
// 等同于在abi.encode编码结果前加上了4字节的函数选择器

function encodeWithSignature() public view returns(bytes memory result) {
    result = abi.encodeWithSignature("foo(uint256,address,string,uint256[2])", x, addr, name, array);
}
```

## abi.encodeWithSelector

```solidity

// 与abi.encodeWithSignature功能类似，只不过第一个参数为函数选择器，为函数签名Keccak哈希的前4个字节。

function encodeWithSelector() public view returns(bytes memory result) {
    result = abi.encodeWithSelector(bytes4(keccak256("foo(uint256,address,string,uint256[2])")), x, addr, name, array);
}

```

## abi.decode

<!-- abi.decode用于解码abi.encode生成的二进制编码，将它还原成原本的参数。 -->

```solidity
function decode(bytes memory data) public pure returns(uint dx, address daddr, string memory dname, uint[2] memory darray) {
    (dx, daddr, dname, darray) = abi.decode(data, (uint, address, string, uint[2]));
}
```

## ABI的使用场景

1. 在合约开发中，ABI常配合call来实现对合约的底层调用。

    ```solidity
    bytes4 selector = contract.getValue.selector;

    bytes memory data = abi.encodeWithSelector(selector, _x);
    (bool success, bytes memory returnedData) = address(contract).staticcall(data);
    require(success);

    return abi.decode(returnedData, (uint256));
    ```

2. ethers.js中常用ABI实现合约的导入和函数调用。

    ```JavaScript
    const wavePortalContract = new ethers.Contract(contractAddress, contractABI, signer);
    /*
        * Call the getAllWaves method from your Smart Contract
        */
    const waves = await wavePortalContract.getAllWaves();
    ```

3. 对不开源合约进行反编译后，某些函数无法查到函数签名，可通过ABI进行调用。

- 0x533ba33a() 是一个反编译后显示的函数，只有函数编码后的结果，并且无法查到函数签名

    ```solidity
    bytes memory data = abi.encodeWithSelector(bytes4(0x533ba33a));

    (bool success, bytes memory returnedData) = address(contract).staticcall(data);
    require(success);

    return abi.decode(returnedData, (uint256));
    ```
