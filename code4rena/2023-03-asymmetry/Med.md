
## [M  01]   ETH will be sucked in SafETH contract   if user directly send ETHER to SafETH contract 

Contract :
https://github.com/code-423n4/2023-03-asymmetry/blob/main/contracts/SafEth/SafEth.sol

 Let us  assume that a user want to stake  2 ETH   , but he sends  accidently  the ETH directly to SafETH contract. So his ETH will be stucked inside of SafETH contract  .This may  also be problematic  because function unstake() calculate  withdraw like this.

  ```js
      uint256 ethAmountAfter = address(this).balance;
        uint256 ethAmountToWithdraw = ethAmountAfter - ethAmountBefore;
        (bool sent, ) = address(msg.sender).call{value: ethAmountToWithdraw }("");
```
    
  
## Proof of Concept
I created a liitle helper function inside safETH .sol (just to quick view ETH balance  )
```js
  function balanceETHERS() public view returns(uint256) {
        return  address(this).balance;
    }

```



```js
  describe("DIRECT DEPLOSIT ", function () {

    it("User can loss his ETH if deposited directly", async function () {
      const startingBalance = await user.getBalance();
      console.log(
        "USR BALANCE ; %s",
        ethers.utils.formatEther(startingBalance)
      );
      const txHash = await user.sendTransaction({
        to: safEthProxy.address,
        value: ethers.utils.parseEther("2.0"),
      });

       await txHash.wait();
      const fbalance = await user.getBalance();
      console.log("AFTER TX USR BALANCE :%s ", ethers.utils.formatEther(fbalance));
      const safPrxoyAddressBalance = await safEthProxy.balanceETHERS();
      console.log("SafETHPROXY balance: %s ", safPrxoyAddressBalance);
      expect(safPrxoyAddressBalance > 0).eq(true);

    });

  });
```

```bash
    DIRECT DEPLOSI
    
 DIRECT DEPLOSIT 
USR BALANCE ; 10000.0 ETH
CALL safETH(0xa9d0fb5837f9c42c874e16da96094b14af0e2784).<UnknownFunction>{value: 9444.732965739290427392}(input: 0x, ret: 0x)
   DELEGATECALL SafEth(0x1eb5c49630e08e95ba7f139bcf4b9ba171c9a8c7).<UnknownFunction>(input: 0x, ret: 0x)
AFTER TX USR BALANCE : 9997.999845852043319574  ETH
CALL safETH(0xa9d0fb5837f9c42c874e16da96094b14af0e2784).balanceETHERS() => (2000000000000000000)
   DELEGATECALL SafEth(0x1eb5c49630e08e95ba7f139bcf4b9ba171c9a8c7).balanceETHERS() => (2000000000000000000)
SafETHPROXY balance: 2.0 ETH
      ✓ User can loss his ETH if deposited directly

```

**Recommended mitigation steps** 
Add a function with a   onlyOwner modifier  to withdraw  ETH to an `` _address `` with a time lock this also have a   risk if the owner is compremised.   
