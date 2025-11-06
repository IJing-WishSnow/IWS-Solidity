# call & deegatecall

## call

目标合约地址.call{value:发送数额, gas:gas数额}(字节码);
其中字节码利用结构化编码函数abi.encodeWithSignature获得.

当用户A通过合约B来call合约C的时候，执行的是合约C的函数，上下文(Context，可以理解为包含变量和状态的环境)也是合约C的：  
msg.sender是B的地址，并且如果函数改变一些状态变量，产生的效果会作用于合约C的变量上。

### 目标合约

```solidity
contract OtherContract {
    uint256 private _x = 0; // 状态变量x
    // 收到eth的事件，记录amount和gas
    event Log(uint amount, uint gas);
    
    fallback() external payable{}

    // 返回合约ETH余额
    function getBalance() view public returns(uint) {
        return address(this).balance;
    }

    // 可以调整状态变量_x的函数，并且可以往合约转ETH (payable)
    function setX(uint256 x) external payable{
        _x = x;
        // 如果转入ETH，则释放Log事件
        if(msg.value > 0){
            emit Log(msg.value, gasleft());
        }
    }

    // 读取x
    function getX() external view returns(uint x){
        x = _x;
    }
}
```

### 调用目标合约

```solidity
// 定义Response事件，输出call返回的结果success和data
event Response(bool success, bytes data);
function callSetX(address payable _addr, uint256 x) public payable {
    // call setX()，同时可以发送ETH
    (bool success, bytes memory data) = _addr.call{value: msg.value}(
        abi.encodeWithSignature("setX(uint256)", x)
    );

    emit Response(success, data); //释放事件
}
function callGetX(address _addr) external returns(uint256){
    // call getX()
    (bool success, bytes memory data) = _addr.call(
        abi.encodeWithSignature("getX()")
    );

    emit Response(success, data); //释放事件
    return abi.decode(data, (uint256));
}
function callNonExist(address _addr) external{
    // call 不存在的函数
    (bool success, bytes memory data) = _addr.call(
        abi.encodeWithSignature("foo(uint256)")
    );

    emit Response(success, data); //释放事件
}
```

## delegatecall

当用户A通过合约B来delegatecall合约C的时候，执行的是合约C的函数，但是上下文仍是合约B的：  
msg.sender是A的地址，并且如果函数改变一些状态变量，产生的效果会作用于合约B的变量上。  

一个投资者（用户A）把他的资产（B合约的状态变量）都交给一个风险投资代理（C合约）来打理。执行的是风险投资代理的函数，但是改变的是资产的状态。

- 注意：delegatecall有安全隐患，使用时要保证当前合约和目标合约的状态变量存储结构相同，并且目标合约安全，不然会造成资产损失。

目前delegatecall主要有两个应用场景：

1. 代理合约（Proxy Contract）：将智能合约的存储合约和逻辑合约分开：代理合约（Proxy Contract）存储所有相关的变量，并且保存逻辑合约的地址；所有函数存在逻辑合约（Logic Contract）里，通过delegatecall执行。当升级时，只需要将代理合约指向新的逻辑合约即可。

2. EIP-2535 Diamonds（钻石）：钻石是一个支持构建可在生产中扩展的模块化智能合约系统的标准。钻石是具有多个实施合约的代理合约。

### delegatecall例子

```solidity
// 调用结构：你（A）通过合约B调用目标合约C。

// 被调用的合约C
// 我们先写一个简单的目标合约C：有两个public变量：num和sender，分别是uint256和address类型；有一个函数，可以将num设定为传入的_num，并且将sender设为msg.sender。

// 被调用的合约C
contract C {
    uint public num;
    address public sender;

    function setVars(uint _num) public payable {
        num = _num;
        sender = msg.sender;
    }
}

// 发起调用的合约B
// 首先，合约B必须和目标合约C的变量存储布局必须相同 —— 即存在两个 public 变量且变量类型顺序为 uint256 和 address

// 注意： 变量名称可以不同

contract B {
    uint public num;
    address public sender;
}

// 接下来，分别用call和delegatecall来调用合约C的setVars函数，更好的理解它们的区别。
// callSetVars函数通过call来调用setVars。它有两个参数_addr和_num，分别对应合约C的地址和setVars的参数。

// 通过call来调用C的setVars()函数，将改变合约C里的状态变量
function callSetVars(address _addr, uint _num) external payable{
    // call setVars()
    (bool success, bytes memory data) = _addr.call(
        abi.encodeWithSignature("setVars(uint256)", _num)
    );
}

// 而delegatecallSetVars函数通过delegatecall来调用setVars。与上面的callSetVars函数相同，有两个参数_addr和_num，分别对应合约C的地址和setVars的参数。

// 通过delegatecall来调用C的setVars()函数，将改变合约B里的状态变量
function delegatecallSetVars(address _addr, uint _num) external payable{
    // delegatecall setVars()
    (bool success, bytes memory data) = _addr.delegatecall(
        abi.encodeWithSignature("setVars(uint256)", _num)
    );
}
```
