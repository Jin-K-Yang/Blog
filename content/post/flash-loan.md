---
title: "無須抵押品就可以貸款？利用 Aave 實作閃電貸（上）"
date: 2023-03-03T11:57:14Z
draft: true
coverImage: "https://images.unsplash.com/photo-1521897258701-21e2a01f5e8b?ixlib=rb-4.0.3&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=3600"
---

在 Defi 的領域中，閃電貸扮演著一個非常重要的角色，他是一把雙面刃，本文將分成上下兩篇分別介紹閃電貸以及利用去中心化借貸平台 Aave 來實作閃電貸智能合約。

![https://images.unsplash.com/photo-1521897258701-21e2a01f5e8b?ixlib=rb-4.0.3&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=3600](https://images.unsplash.com/photo-1521897258701-21e2a01f5e8b?ixlib=rb-4.0.3&q=80&fm=jpg&crop=entropy&cs=tinysrgb&w=3600)

## 甚麼是閃電貸？

閃電貸是一種無需抵押品便可以獲得貸款的方式，而且貸款沒有限制，這是利用以太坊上的交易機制來達到的。**在同一筆交易中借款還款**，這樣就不會改變區塊的 state ，以區塊的角度來看，**你的餘額完全沒有改變**，但其實你已經借了幾百萬美金又還款了，這就是閃電貸的基本原理。

但你可能會問，如果我沒有還款不就可以拿走這些錢了嗎？以太坊的交易機制保證一筆交易要馬全部執行，要馬完全不執行。如果借款人在交易結束前沒有還款，那麼交易就會被 revert ，回到上一個 state ，也就是說**借款從來沒有發生過**，因此不會有不還款交易就完成的情況發生。使用者只會損失這筆交易執行到一半的 gas fee 而已。

## 常見的閃電貸使用方式

## 閃電貸攻擊

- 如何避免

# References

- [https://www.moonpay.com/learn/defi/defi-flash-loans#what-are-flash-loans](https://www.moonpay.com/learn/defi/defi-flash-loans#what-are-flash-loans)