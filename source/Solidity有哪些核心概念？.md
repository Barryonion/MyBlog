# Solidity有哪些核心概念？



# 数据类型

| 类型                | 关键字          | 简介                                                         | 声明方式                                                 |
| ------------------- | --------------- | ------------------------------------------------------------ | -------------------------------------------------------- |
| Boolean             | bool            | 布尔类型，值为true或false                                    | bool                                                     |
| Interger            | int、uint等     | 整形，默认为256位                                            | int clientCounter = 6;                                   |
| Fixed point         | fixed、ufixed   | 小数类型，以(u)fixedMxN形式定义，M是中位数，N是小数点后位数。M必须是8的倍数且小于256 | (u)fixed8x2 = 123456.02;                                 |
| Address             | address         | 以太坊地址，20个字节，包含balance、transfer等成员方法        | address a;                                               |
| Byte array(fixed)   | bytes1至bytes32 | 固定大小的字节数组                                           | bytes1 ba;<br />bytes32 ba32;                            |
| Byte array(dynamic) | bytes或string   | 可变长度的字节数组                                           | bytes bc;<br />string str                                |
| Enum                | enum            | 枚举类型                                                     | enum NAME  {LABEL1,LABEL2,···}                           |
| Struct              | struct          | 结构体，聚合其他类型的自定义类型                             | struct NAME  {TYPE1 VARIABLE1;  TYPE2 VARIABLE2; ······} |
| Mapping             | mapping         | 映射类型，保存键值对                                         | mapping(KEY_TYPE => VALUE_TYPE) NAME                     |



# 预定义的全局变量

## msg对象

msg对象与交易/信息调用的上下文相关，即与触发当前合约代码执行的调用相关，该调用即可能是由外部以太坊账户触发，也可能是由已经部署的智能合约触发。

| 属性名称   | 简介                                                        |
| ---------- | ----------------------------------------------------------- |
| msg.sender | 发起本次调用的地址，外部账户地址或合约地址                  |
| msg.value  | 本次调用所转移的以太币数量（以wei为单位，1ETH=10^18 wei）   |
| msg.gas    | 剩余的gas数量                                               |
| msg.data   | 本次调用包含的数据                                          |
| msg.sig    | 本次调用中所包含的数据的头4个字节，通常用来作为方法选择器。 |

注意，msg对象中包含的是调用方的信息。倘若在一条调用链路中涉及多个合约，例如A调用B，B调用C，C调用D，那么msg对象会根据调用链路的执行情况不断地更新，由A更新到B再更新到C。



## 交易环境

tx对象提供了访问交易信息的方式

| 名称        | 简介              |
| ----------- | ----------------- |
| tx.gasprice | 发起交易的gas价格 |
| tx.origin   | 发起交易的EOA地址 |



## 区块环境

block对象包含当前区块的信息

| 名称            | 简介                                    |
| --------------- | --------------------------------------- |
| block.coinbase  | 获得交易费用和打包奖励的地址            |
| block.difficuty | 当前区块打包的难度系数                  |
| block.gaslimit  | 当前区块所包含的交易中可花费的最大gas量 |
| block.number    | 当前的区块高度                          |
| block.timestamp | 当前区块的时间戳                        |



## 地址对象

address对象，用来表示外部账户EOA或内部合约。

| 名称                      | 简介                                                         |
| ------------------------- | ------------------------------------------------------------ |
| address.balance           | 地址的以太币余额。例如，当前合约的余额可表示为：address(this).balance |
| address.transfer(amount)  | 把指定amount数量的以太币转移到当前地址，一旦出现错误，立即抛出异常 |
| address.send(amount)      | 与transfer功能类似，不同之处在于，一旦报错，将返回false给调用方 （提示：需检查返回值） |
| address.call(payload)     | 谨慎使用，更低级别的调用方法，可以发起对任意一个方法的调用，一旦报错，将返回false给调用方 （提示：需检查返回值） |
| address.callcode(payload) | 谨慎使用，更低级别的调用方法，与上述address.call(payload)类似，不同之处在于address会由合约代替，一旦报错，将返回false给调用方 （提示：需检查返回值） |
| address.delegatecall()    | 谨慎使用，与上述address.callcode(payload)类似，不同之处在于，当前合约可以访问整个msg对象的内容，一旦报错，将返回false给调用方 （提示：需检查返回值） |



# 内置函数

| 名称                             | 简介                                                       |
| -------------------------------- | ---------------------------------------------------------- |
| addmod,mulmod                    | 用于模数加法和乘法运算。例如，addmod(x,y,k) 计算 (x+y) % k |
| keccak256,sha256,sha3,ripemd160  | 以不同的方法来计算哈希值                                   |
| ecrecover                        | 从签名中恢复用于签署信息的地址                             |
| selfdestrunct(recipient_address) | 删除当前合约，将账户中剩余的以太币发送至收件人地址         |
| this                             | 当前执行的合约账户的地址                                   |



# 函数

合约中定义的函数可以被EOA发起的交易或另一个合约来调用。

定义一个函数的语法如下：

```solidity
function FunctionName([parameters]) {public|private|internal|external} [pure|constant|view|payable] [modifiers] [returns (return types)]
```

| 名称         | 解释                                                         |
| ------------ | ------------------------------------------------------------ |
| function     | 用来声明函数的关键字                                         |
| FunctionName | 函数名，用来唯一标识一段代码片段。没有函数名的函数是fallback函数，它用来执行默认的操作 |
| parameters   | 函数入参                                                     |
| public       | 默认的函数可见性级别，此类函数可以被其他合约或EOA交易或方法所在的合约来调用 |
| external     | 除了不能隐式地调用方法所在的合约代码之外，其余与public类似，若要调用方法所在的合约代码，需要结合this关键字 |
| internal     | 可在该方法定义的合约或继承该合约的子合约中调用               |
| private      | 只能在定义该方法的合约中被调用                               |
| view         | 修饰的方法不会更新状态                                       |
| pure         | 修饰的方法不会读写存储的变量                                 |
| payable      | 修饰的方法可以接收以太币                                     |



# 合约的构造和销毁

每一个智能合约只会进行一次初始化，它的初始化过程发生在构造函数之中，一旦将合约部署到EVM，构造函数将会首先被执行。在较早版本的Solidity中，构造函数的命名必须要与合约名称保持一致：

```solidity
contract MyContract{
        function MyContract(){
        
        }
}
```

但是，倘若变更合约名称时，不小心遗漏了对构造函数名的变更，那么该构造函数将会变成一个普通函数。所以在Solidity v0.4.22之后的版本中，新增了关键字constructor，专门用来声明构造函数

```solidity
pragma ^0.4.22
contract MyContract{
        function MyContract(){
        
        }
}
```

已经部署的智能合约不能够再更改，但是可以被销毁，前提是在合约中声明了销毁的方法，销毁方法的声明如下：

```solidity
selfdestruct(address recipient);
```

其中的入参是一个地址，一旦将当前合约销毁，那么合约中剩余的以太币将会被发送到该地址表示的账户之中。



# 函数修饰符

函数修饰符是在声明函数时使用的，一般包含了程序的条件判断逻辑，对特定的代码片段进行访问控制。

例如以下代码声明了函数修饰符onlyManager：

```solidity
modifier onlyManager() {
        require(msg.sender == manager, "Only manager can call this!");
        _;
    }
```

只有管理员的账户地址才会执行后续的逻辑。其中，占位符"_"代表着被修饰的代码逻辑，例如：

```solidity
function sendInterest() public payable onlyManager{
        for(uint i=0;i<clients.length;i++){
            address initialAddress = clients[i].client_address;
            uint lastInterestDate = interestDate[initialAddress];
            if(now < lastInterestDate + 10 seconds){
                revert("It's just been less than 10 seconds!");
            }
            payable(initialAddress).transfer(1 ether);
            interestDate[initialAddress] = now;
        }
    }
```

以上代码的方法sendInterest被onlyManager修饰，那么占位符"_"代表整个for循环中的代码片段。

函数修饰符可以将重复使用的判断条件收敛起来，提高代码的整洁度和安全性。

变量可见关系：在定义的函数修饰符中，可以访问那些被它修饰的函数可见的变量。但是，反过来，被函数修饰符修饰的函数无法访问函数修饰符中可见的变量



# 继承

合约之间可以存在继承关系，一旦实现继承，那么子合约将会获得父合约的所有成员，并且它还支持多继承，即一个子合约可以继承多个父合约，继承使用到is关键字。例如：

```solidity
contract Child is Parent1, Parent2{
	···
}
```

通过继承的特性，可以提升合约代码的模块性、可扩展、复用性。



# 附录(学练习资料)

https://capturetheether.com/challenges/

https://ethernaut.openzeppelin.com/

https://www.damnvulnerabledefi.xyz/

https://github.com/fvictorio/evm-puzzles

https://github.com/foundry-rs/foundry

https://cryptozombies.io/





# 参考资料：

https://www.amazon.com/Mastering-Ethereum-Building-Smart-Contracts/dp/1491971940/
https://betterprogramming.pub/developing-a-smart-contract-by-using-remix-ide-81ff6f44ba2f

