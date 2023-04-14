---
title: "無須抵押品就可以貸款？利用 Aave 實作閃電貸智能合約（下）"
author: Ken
date: 2023-04-14T07:54:06Z
draft: false
tags: [
    "Blockchain",
    "Ethereum",
    "Defi",
    "Security",
    "Coding"
]
coverImage: "https://images.unsplash.com/photo-1521897258701-21e2a01f5e8b?ixlib=rb-4.0.3&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=3600"
---
上一篇我們介紹了甚麼是閃電貸，並且介紹了閃電貸的使用場景，這篇我們將介紹閃電貸的合約是如何實現的，並且用 Aave 實作出來。

![https://images.unsplash.com/photo-1521897258701-21e2a01f5e8b?ixlib=rb-4.0.3&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=3600](https://images.unsplash.com/photo-1521897258701-21e2a01f5e8b?ixlib=rb-4.0.3&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=3600)
<center>Photo by  <a href="https://unsplash.com/@jasondeblooisphotography?utm_source=medium&utm_medium=referral">Jason D</a> on <a href="https://unsplash.com/?utm_source=medium&utm_medium=referral">Unsplash</a></center>

## 閃電貸流程

首先，我們先來看看 Aave 閃電貸的流程是甚麼

![https://i.imgur.com/trz4qzn.png](https://i.imgur.com/trz4qzn.png)

1. 對你的合約發起一個交易，你的合約會呼叫 Aave pool 合約裡面的 `FlashLoanSimple()` 並指定這次閃電貸要借的 token 數量。
2. Aave pool 合約執行 `FlashLoanSimple()` 轉移要借出的 token 到你的合約，並且執行你的合約中的 `executeOperation()`。
3. 現在你的合約中擁有從 Aave 借來的貸款，並且可以做你想做的任何事情，等到 `executeOperation()` 執行成功後，Aave pool 合約會從你的合約中把應該要還的錢加上費用一同償還給 Aave。
4. 以上所有事情都發生在同一個 transaction 中，因此也會在同一個 block。

## Aave 閃電貸合約簡介

接下來我們透過程式碼來實際看看 Aave 的 `FlashLoanSimple()` 流程，以下只會列出跟閃電貸有關的程式碼，有興趣的讀者可以到他們的 [GitHub](https://github.com/aave/aave-v3-core) 觀看完整的程式碼。

- 以下是 [`FlashLoanSimple()`](https://github.com/aave/aave-v3-core/blob/master/contracts/protocol/pool/Pool.sol#L427) 的完整程式碼，可以看到最下面一行的 `executeFlashLoanSimple()` 才是真正在執行閃電貸的程式碼。

```solidity
// Pool.sol

function flashLoanSimple(
  address receiverAddress,
  address asset,
  uint256 amount,
  bytes calldata params,
  uint16 referralCode
) public virtual override {
  DataTypes.FlashloanSimpleParams memory flashParams = DataTypes.FlashloanSimpleParams({
    receiverAddress: receiverAddress,
    asset: asset,
    amount: amount,
    params: params,
    referralCode: referralCode,
    flashLoanPremiumToProtocol: _flashLoanPremiumToProtocol,
    flashLoanPremiumTotal: _flashLoanPremiumTotal
  });
  FlashLoanLogic.executeFlashLoanSimple(_reserves[asset], flashParams);
}
```

- 接著我們繼續追蹤 [`executeFlashLoanSimple()`](https://github.com/aave/aave-v3-core/blob/master/contracts/protocol/libraries/logic/FlashLoanLogic.sol#L179) 看看他在做甚麼，以下程式 碼只節錄重要部分，可以看到在 `require()` 裡面的第一個參數是 `receiver.executeOperation()`，`receiver` 就是你的合約，他會執行你合約中的 `executeOperation()`，如果回傳的是 `true`，代表你的合約已經透過閃電貸做完你想做的事情，接下來會需要償還貸款，那就是 `_handleFlashLoanRepayment()` 所處理的。

```solidity
// FlashLoanLogic.sol

function executeFlashLoanSimple(
  DataTypes.ReserveData storage reserve,
  DataTypes.FlashloanSimpleParams memory params
) external {

  // ...

  require(
    receiver.executeOperation(
      params.asset,
      params.amount,
      totalPremium,
      msg.sender,
      params.params
    ),
    Errors.INVALID_FLASHLOAN_EXECUTOR_RETURN
  );

  _handleFlashLoanRepayment(
    reserve,
    DataTypes.FlashLoanRepaymentParams({
      asset: params.asset,
      receiverAddress: params.receiverAddress,
      amount: params.amount,
      totalPremium: totalPremium,
      flashLoanPremiumToProtocol: params.flashLoanPremiumToProtocol,
      referralCode: params.referralCode
    })
  );
}
```

- 在 [`_handleFlashLoanRepayment()`](https://github.com/aave/aave-v3-core/blob/master/contracts/protocol/libraries/logic/FlashLoanLogic.sol#L223) 中，我們一樣只看最重要的部分，Aave 會從你的合約中將一開始借出的 token 數量，加上使用閃電貸的服務費用，一次轉回來。在這個步驟中，如果你的合約 token 餘額不足，或是 allowance 給的不夠，交易就會被 revert。到這邊整個閃電貸才算是完成。

```solidity
// FlashLoanLogic.sol

function _handleFlashLoanRepayment(
  DataTypes.ReserveData storage reserve,
  DataTypes.FlashLoanRepaymentParams memory params
) internal {

  // ...

  IERC20(params.asset).safeTransferFrom(
    params.receiverAddress,
    reserveCache.aTokenAddress,
    amountPlusPremium

  // ...

);
```

## 合約實作

知道了閃電貸的流程以及 Aave 合約中是如何實作的之後，我們就要開始寫一個自己的合約來呼叫 Aave 的閃電貸 function。

- 首先，我們要先將會用到的 Aave 合約下載下來。

```bash
npm i @aave/core-v3
```

- 接著引入會使用到的合約並且繼承 `FlashLoanSimpleReceiverBase`。

```solidity
// FlashLoan.sol
// SPDX-License-Identifier: MIT
pragma solidity 0.8.10;

import {FlashLoanSimpleReceiverBase} from "@aave/core-v3/contracts/flashloan/base/FlashLoanSimpleReceiverBase.sol";
import {IPoolAddressesProvider} from "@aave/core-v3/contracts/interfaces/IPoolAddressesProvider.sol";
import {IERC20} from "@aave/core-v3/contracts/dependencies/openzeppelin/contracts/IERC20.sol";

contract FlashLoan is FlashLoanSimpleReceiverBase {}
```

- 實作 `constrctor()`，這邊需要 `_addressProvider` 作為 input 是因為 Aave 的借貸池子是可變動的，所以為了要拿到現在可以借貸的池子，Aave 提供了 PoolAddressesProvider 可供查詢，這個參數就是那個合約的地址，可以從 Aave 的 [Doc](https://docs.aave.com/developers/deployed-contracts/v3-mainnet/ethereum-mainnet) 查到。

```solidity
constructor(address _addressProvider) FlashLoanSimpleReceiverBase(IPoolAddressesProvider(_addressProvider)) {
  owner = payable(msg.sender);
}
```

- 實作 `executeOperation()` 給 Aave 合約呼叫。

```solidity
function executeOperation(
  address asset,
  uint256 amount,
  uint256 premium,
  address initiator,
  bytes calldata params
) external override returns (bool) {
  // Logic you want to implement goes here

  // Approve the Pool contract allowance to *pull* the owed amount
  uint256 amountOwed = amount + premium;
  IERC20(asset).approve(address(POOL), amountOwed);

  return true;
}
```

- 實作 `requestFlashLoan()` 給 EOA 呼叫，並且呼叫 Aave 的閃電貸 function。

```solidity
function requestFlashLoan(address _token, uint256 _amount) public {
  address reciverAddress = address(this);
  address asset = _token;
  uint256 amount = _amount;
  bytes memory params = "";
  uint16 referralCode = 0;

  POOL.flashLoanSimple(reciverAddress, asset, amount, params, referralCode);
}
```

- 別忘了加上 `withdraw()`，否則在合約裡面的錢是拿不出來的。

```solidity
function withdraw(address _tokenAddress) external onlyOwner {
  IERC20 token = IERC20(_tokenAddress);
  token.transfer(msg.sender, token.balanceOf(address(this)));
}
```

以上就是利用 Aave 實作閃電貸合約的所有細節，有興趣的讀者可以到[這裡](https://github.com/Jin-K-Yang/Flash-loan)查看完整的 repo。

## Conclusion

經過之前的介紹，我們已經對閃電貸有了初步的認識，在這篇文章中我們更深入地了解了閃電貸的合約實現方式以及使用 Aave 實作的例子。希望大家能夠注意任何有可能利用閃電貸所引發的漏洞，同時也能善用區塊鏈的特性，利用閃電貸激盪出與傳統金融截然不同的點子。

## References

- [Aave Flash Loans Docs](https://docs.aave.com/developers/v/2.0/guides/flash-loans)