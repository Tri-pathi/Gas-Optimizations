# Gas Optimizations in Solidity

In this tutorial, we will learn about some of the gas optimization techniques in Solidity.

Let's get started, this is one of the most requested articles 👀


## Tips and Tricks


# Packing your variables in Solidity

If you remember we talked about storage slots in one of our previous levels. Now the interesting point in solidity if you remember is that each storage slot is 32 bytes.

Now storage can be optimized which will further mean gas optimization when you deploy your smart contract if you pack your variables correctly.

Packing your variables means that you pack or put together variables of smaller size so that they collectively form `32 bytes`. For example, you can pack 32 `uint8` into one storage slot but for that to happen it is important that you declare them consecutively because the order of declaration of variables matters in solidity.

Given two code samples:

```solidity
uint8 num1;
uint256 num2;
uint8 num3;
uint8 num4;
uint8 num5;
```
```solidity
uint8 num1;
uint8 num3;
uint8 num4;
uint8 num5
uint256 num2;
```

The second one is better because in the second one solidity compiler will put all the `uint8`'s in one storage slot but in the first case it will put `uint8 num1` in one slot but now the next one it will see is a `uint256` which is in itself requires 32 bytes cause `256/8 bits = 32 bytes` so it cant be put in the same storage slot as `uint8 num1` so now it will require another storage slot. After that `uint8 num3, num4, num5` will be put in another storage slot. Thus the second example requires 3 storage slots as compared to the first example which requires only two storage slots.

It's also important to note that elements in `Memory` and `Calldata` cannot be packed and are not optimized by solidity's compiler.


## Storage VS Memory

Changing storage variables requires more gas than variables in memory.
It's better to update storage variables at the end after all the logic has already been implemented.

So given two samples of code 

```solidity=
{
    uint public counter = 0;
    
    function count() {
        for(uint i = 0; i< 10; i++) {
            counter++;
        }
    }
    
}
```

```solidity
{
    uint public counter = 0;
    
    function count() {
        uint copyCounter;
        for(uint i = 0; i< 10; i++) {
            copyCounter++;
        }
        counter = copyCounter;
    }
    
}
```

The second sample of code is more gas optimized because we are only writing to the storage variable `counter` only once as compared to the first sample where we were writing to storage in every iteration

## Fixed length and Variable-length variables

We talked about how fixed length and variable length variables are stored. Essentially variable-length variables are stored in a stack whereas variable-length variables are stored in a heap. 

Essentially why this happens is because in a stack you exactly know where to find a variable and its length whereas in a heap there is an extra cost of traversing given the variable nature of the variable

So if you can make your variables fixed size, it's always good for gas optimizations

Given two examples of code:

```solidity
string public text = "Hello";
uint[] public arr;
```

```solidity
bytes32 public text = "Hello";
uint[2] public arr;
```

The second example is more gas optimized because all the variables are of fixed length.

# External, Internal, and Public functions

Calling external functions in solidity is very gas-intensive, its better you call one function and extract all data from it than call multiple external functions

Also when your contract is creating functions that will only be called externally it means the contract itself cant call these functions. Its better you use the `external` keyword instead of `public` because all the input variables in `public` functions are copied to memory which costs gas whereas for `external` functions input variables are stored in `calldata` which is a special data location used to store function arguments and it requires less gas to store in calldata than in memory

The same principle applies as to why it's cheaper to call `internal` functions rather than `public` functions. This is because when you call `internal` functions the arguments are passed as references of the variables and are not again copied into memory but that doesn't happen in the case of `public` functions.


## Function modifiers

This is a fascinating one because a few weeks ago, I was debugging this error from one of our students and he/she was experiencing the error “Stack too deep”. This usually happens when you declare a lot of variables in your function and the available stack space for that function is no longer available. Now even after moving a lot of the require statements in the `modifier` it wasn't helping because function modifiers use the same stack as the function on which they are put. To solve this issue we used an `internal` function inside the `modifier` because `internal` functions don't share the same restricted stack as the `original function` but `modifier` does.


## Use libraries

Libraries are stateless contracts that don't store any state. Now when you call a public function of a library from your contract, the bytecode of that function doesn't get deployed with your contract, and thus you can save some gas costs. For example, if you contract has functions to sort or to do maths etc. You can put them in a library and then call these library functions to do the maths or sorting for your contract. To read more about libraries follow this [link](https://jeancvllr.medium.com/solidity-tutorial-all-about-libraries-762e5a3692f9)

## Other general tips and tricks

If you are using (||) or (&&) it's better to write your conditions in such a way so that the least functions/variable values are executed or retrieved in order to determine if the entire statement is true or false.

Freeing up storage space leads to gas refunds in solidity. So if you don't need a variable, consider using the `delete` keyword for some gas refunds

Make sure that the error strings in your require statements are of very short length, the more the length of the string, the more gas it will cost.

```solidity=
require(counter >= 100, "NOT REACHED"); //good
require(balance >= amount, "Counter is still to reach the value greater than or equal to 100, ............................................";
```
The first requirement is more gas optimized than the second one.

----

Thank you all for staying tuned to this article 🚀 Hope you liked it :)


## References

- Mudit Gupta - [Gas Optimizations Tips](https://mudit.blog/solidity-gas-optimization-tips/) and [Gas Optimizations Tips Part - 2](https://mudit.blog/solidity-tips-and-tricks-to-save-gas-and-reduce-bytecode-size/)